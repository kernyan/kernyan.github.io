---
author: kernyan
comments: true
date: 2019-01-11 01:27:06+00:00
layout: post
link: http://kernyan.com/2019/01/11/what-passing-by-value-actually-means-to-the-cpu/
slug: what-passing-by-value-actually-means-to-the-cpu
title: What Passing by Value Actually Means to the CPU
categories:
- C/C++
- CPU
---




_Taking a look at pass-by-value and pass-by-reference from the perspective of the computer_







In C, there are three ways of passing a variable to a function, (1) by-value, (2) by-reference, and (3) by-pointer. 







The compiler cares about the distinction between (2) and (3), but not the CPU. Thus, we are going to consider (2) and (3) as the same case. You can convince yourself that they are indeed the same by inspecting the disassembly below.





![](/assets/images/2019_01_ptr_ref.png)





Back to the more interesting discussion of pass-by-value vs pass-by-reference. The following code snippet shows a function call which takes the same variable as parameter





```c
struct Four64Bits
{    
    long long i1;
    long long i2;
    long long i3;
    long long i4;  
};  

void ByReference (Four64Bits &in) 
{
}

void ByValue (Four64Bits in) 
{
}

int main()
{
    Four64Bits Var;

    ByReference(Var);    
    ByValue(Var);

    return 0;
}
```





The two function calls do exactly nothing, and they vary only in the parameter type. Also, you might have heard that passing a parameter by reference instead of by value would save you a copy. To concretely see what it means by "saving a copy", let's take a look at [godbolt](https://godbolt.org/)'s disassembly again,





![](/assets/images/2019_01_value_ref.png)





**On ByReference**







Right side's line 18 ```lea rax, [rbp-32]``` is taking the memory location of **Var**. The memory location is ```[rbp-32]``` as local variables are allocated on the stack with a negative offset of its size (and Four64Bits is 32 bytes). Line 19 ```mov rdi, rax``` is moving the memory location of **Var** to register ```rdi```, which you can then see in line 4, where it is being used in the ByReference instructions. What's most important to notice is that we only needed a 64-bit memory location to pass a variable of 4*64 bits in size.  








**On ByValue**







On the other hand, right side's line 21 to 24 of ```push QWORD PTR [rbp-8x]``` shows that the CPU had to execute 4 instructions just to pass **Var** to the ByValue function. The ```push QWORD``` means that the CPU is copying 8 bytes of data to the stack. Thus we see that four ```push QWORD``` instructions  are required to fully copy a 32 byte variable. 







Takeaway is that passing by value means the CPU is physically copying all the bits of the variable instead of just the address of its memory location.  




















  




