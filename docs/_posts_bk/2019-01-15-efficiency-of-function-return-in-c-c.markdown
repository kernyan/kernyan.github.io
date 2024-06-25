---
author: kernyan9
comments: true
date: 2019-01-15 20:44:48+00:00
layout: post
link: http://kernyan.com/2019/01/15/efficiency-of-function-return-in-c-c/
slug: efficiency-of-function-return-in-c-c
title: Efficiency of Function Return in C/C++
wordpress_id: 1764
categories:
- C/C++
---




_Function return mechanism in assembly. Impact of Return Value Optimization._  








In the C code snippet below, 






    
    <code>[code language=c]
    UserObject Foo()
    {
        UserObject Local = UserObject();
        return Local;
    }
    
    int main()
    {
        UserObject O = Foo();
    }
    [/code]</code>







[code]Foo()[/code] has a class return value, which is the local variable in line 3. 







Let's start with the behavior without Return Value Optimization (RVO). As typical for local variables in C, when the instruction jumps to Foo(), the CPU will 1) allocate the local variable "Local" on the stack, and 2) call "Local"'s default constructor on the allocated stack space. At the end of the function call, the returned value is passed back to the caller by keeping the memory address of "Local" in the EAX/RAX register. Subsequently, with the control returning to the main function, the CPU would call "O"'s copy (or move constructor if there's one) with the returned value as argument.







The key thing to note here is that "O" is constructed by first having the CPU construct "Local" in Foo(), and then calling "O"'s copy constructor.







In C++17, the standard now guarantees RVO behavior. This means that instead of allocating on the stack memory for "Local",  the CPU would call UserObject's constructor in Foo() on its caller's variable location (in this case, the memory address of "O"). With this, there's no longer the need for the caller (i.e. main) to call UserObject's copy constructor when Foo() returns. 















