---
author: kernyan
comments: true
date: 2020-12-08 02:46:28+00:00
layout: post
link: http://kernyan.com/2020/12/08/infinite-page-fault-triple-fault-and-general-protection-fault/
slug: infinite-page-fault-triple-fault-and-general-protection-fault
title: Infinite Page Fault, Triple Fault, and General Protection Fault
wordpress_id: 2025
categories:
- C/C++
- OS
---




In KernOS's development, we are at the point of handling page fault exceptions. To do this, I have written the following test code,





```cpp
    
    volatile uint32_t *Mem = (uint32_t*) 0x400000;
    *Mem = 10;
    
    volatile uint32_t *Mem2 = (uint32_t*) 0x400f00;
    *Mem2 = 20;

```





Memory location 0 to 4MB has been identity mapped at kernel initialization (see [virtualmemory.cpp](https://github.com/kernyan/KernOS/blob/master/OS/kernel/arch/x86/virtualmemory.cpp)). Thus we are going to intentionally access memory beyond 4MB to trigger page fault.







The first **Mem** is expected to cause a page fault, while the second **Mem2** being less than 4K apart from the first, should not trigger a page fault; as the latter's page would have been mapped.







To test that the unmapped memory access can 1) trigger a page fault, and 2) be handled by adding a corresponding page table entry to the address' page table, I wrote a crude page fault handler as below.





```cpp
    
    extern "C" void FaultPageHandler()
    {
        uint32_t Fault_Address;
        asm volatile("mov %%cr2, %0" : "=r" (Fault_Address));
    
        if (  Fault_Address >= 4 * MB
           && Fault_Address  < 8 * MB
           )
        {
          VM::MapPageTable(1, VM::kernel_page_directory, VM::pagetable1);
    
          asm volatile( // flush tlb
          "invlpg %0"
          :
          : "m"(*(char*) Fault_Address)
          : "memory"
          );
        }
        else
        {
            kprintf("Page fault handler only works for 4MB - 8MB range for now\n");
            Hang();
        }
    
        kprintf("Page fault handler called\n");
    }
    
```






Booting the kernel, I was expecting to see "Page fault handler called" being logged only once. And sure enough, that's exactly what _did not_ happen.







Instead, I was greeted with an infinite page fault - where the page fault handler code keeps getting called. Initially, I attempted to troubleshoot it by haphazardly attempting whatever comes to mind, like resetting cr3 to the page directory location, flushing (or not flushing) tlb, and dumping registers.







Whatever it is, they didn't work. But at some point, I got a triple fault. And after some more directionless meddling, I got a general protection fault. It's clear that at this point that something is horribly wrong.







However, reasoning the various types of error do give us some clues. An infinite page fault indicates that we are repeatedly accessing unmapped memory; a triple fault indicates that we are encountering a second and third fault while handling the first fault; and a general protection fault indicates that we could be accessing protected memory, or simply executing bytes in memory not designated as instructions.







I thought this could be coming from the page fault handler jumping into unexpected memory location. Inspecting the exception handler, it appears innocent, as it's the exact same wrapper that I used for an interrupt handler. The interrupt handler has been tested to be working on the PIT controller.







The interrupt handler macro below simply wraps the handler call with an _iret_ instruction.





```cpp
    
    #define INTRP_ENTRY(Type)                     \
      extern "C" void Interrupt##Type##Entry();   \
      extern "C" void Interrupt##Type##Handler(); \
      asm(                                        \
      ".globl Interrupt" #Type "Entry \n"         \
      "Interrupt" #Type "Entry:       \n"         \
      "call Interrupt" #Type "Handler \n"         \
      "    iret\n");
    
    INTRP_ENTRY(Timer)

```





Finally, on some google-fu, I chanced upon a comment which said that the CPU would push an addition error code on certain exceptions that was not the case for interrupts.







Thus the exception handler _shouldn't_ be the exact same as the interrupt handler. As it should pop the error code before returning, otherwise _iret_ would be popping the stack content's of [error code, return address, segment registers, flags] into [instruction pointer, segment registers, flags], thoroughly messing up $eip, segment registers, and flags. This explains the infinite page fault, triple fault, and general protection fault that we were running into.







Here's the revised exception handler code that correctly pops the error code.





```cpp
    
    #define FAULT_ENTRY(Type)                   \
        extern "C" void Fault##Type##Entry();   \
        extern "C" void Fault##Type##Handler(); \
        asm(                                    \
        ".globl Fault" #Type "Entry \n"         \
        "Fault" #Type "Entry:       \n"         \
        "    call Fault" #Type "Handler \n"     \
        "    add $0x4, %esp\n"                  \
        "    iret\n");
    
    FAULT_ENTRY(Page)

```





The plan next is to get physical memory info from the bootloader, and organize them into 4KB pages of available page frame for allocation. With this, we can then enhance our page fault handler so as to be capable of allocating virtual addresses beyond 8MB.



