---
author: kernyan
comments: true
date: 2019-08-03 03:46:37+00:00
layout: post
link: http://kernyan.com/2019/08/03/comparison-operator-for-any-user-defined-class-in-c/
slug: comparison-operator-for-any-user-defined-class-in-c
title: Comparison operator for any user defined class in C++
categories:
- C/C++
---




_For fast indexing / lookup without having to specify hash function in C++, we can use memcmp to achieve generic comparison operator for arbitrary class_







When we need a dictionary (or key value pair) structure in C++, a good place to start is to consider std::map, or std::unordered_map. 







std::unordered_map requires that the key be hash-able (i.e., that std::hash<type> be defined), while std::map requires that the key has the less than comparison operator be defined.







std::unordered_map being implemented as a hash table provides an O (1) lookup, while std::map being implemented as a tree provides an O (log n) lookup. Therefore if lookup time is our main concern, we should prefer unordered_map.







However, when the key is a user defined class, defining a good hash function is not straightforward. A good hash function needs several properties, among others, 







  * to be fairly uniform in mapping the domain over the possible range,   
  * range has to be large enough to make collision unlikely,
  * hash algorithm has to be fast and account for arbitrary member layout of the user defined class






When the user defined class has uniform members, a hash function similar to [java's hashCode ](http://javadevnotes.com/java-string-hashcode)implementation is a good reference. The idea roughly works as, 






```cpp    
    struct UserObj 
    { 
        float a; 
        float b; 
        float c; 
    }; 
    
    namespace std 
    { 
        template <> 
        struct hash<UserObj> 
        { 
            size_t operator() (UserObj const & in) const 
            { 
                const size_t n = sizeof(UserObj) / sizeof (float); 
                float *p = (float*) &in; 
                size_t key = 17; 
                for (size_t i = 0; i < n; ++i) { 
                    key = 31 * key + hash<float> ()(*p++); 
                } 
                return key; 
            } 
        }; 
    }
```






However, the hash function above has two limitations, 







  1. size_t is the return type defined by std::hash, and thus our hash template specialization is limited to the range of size_t, which on a 32 bit architecture has a [1 in 10'000 probability of collision](https://preshing.com/20110504/hash-collision-probabilities/) when our container size gets to around 1'000, which is quite unsatisfactory (_and that's assuming that our hash algorithm uniformly spreads the input keys over the 32 bit integer range, which is not always true_), 
  2. the hash function is generic only when all the members of the user defined class are of the same type, otherwise the pointer arithmetic hash algorithm would have to be reviewed each time a new member is added.






What's the alternative then?







If we can live with ```O (log n)``` lookup, then we can get away without the hash function entirely, by using std::map as the container. In exchange for the slower lookup, we no longer have to worry about hash collision. The ```O (log n)``` lookup complexity is a consequence of std::map's implementation, which orders the elements via a binary tree. That's also why we need to define a less than operator.







Now, for a less than operator, all we need is to have a rule that systematically tells us whether an element is less than another element. As long as the rule is systematic, there's no requirement on what rule we use. Knowing this, the simplest (and also most generic way) of comparing two arbitrary element is to compare their bits. Our rule could then be whichever element that has the first different bit being smaller. It doesn't matter if the bits are ordered little or big endian too, because either way, it is still systematic.







This is precisely what the built-in C library function memcmp does,  where







> **`memcmp`**
> 
> *returns value < 0 when the first byte that does not match in both memory blocks has a lower value in ptr1 than in ptr2 (if evaluated as unsigned char values)*
> 
> Source: [cplusplus.com](http://www.cplusplus.com/reference/cstring/memcmp/)







Thus we can write something like,






```cpp    
    bool operator<(UserObj const & lhs, UserObj const & rhs)
    {
        return memcmp(&lhs, &rhs, sizeof(lhs)) < 0;
    }
```







Caveat worth mentioning is that this only works on structure which does not contain pointer or references, since in such a case, memcmp would be comparing the pointer address instead of the content.



