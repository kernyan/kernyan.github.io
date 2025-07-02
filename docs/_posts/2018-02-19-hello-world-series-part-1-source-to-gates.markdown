---
author: kernyan
comments: true
date: 2018-02-19 05:09:11+00:00
layout: post
link: http://kernyan.com/2018/02/19/hello-world-series-part-1-source-to-gates/
slug: hello-world-series-part-1-source-to-gates
title: Source code to machine instruction
categories:
- C/C++
- CPU
---




_Tracing the ubiquitous "Hello World!" as far as we can_







When you compile the following code,





```cpp

int main()
{
  printf ("Hello World!");
  return 0;
}

```










Then the compiler processes the text above into machine code. For now, simply take that the compiler is a program that takes the source file we see above and spits them out in another form that the specific computer architecture that you're running on can operate. The output being an [executable](https://en.wikipedia.org/wiki/Comparison_of_executable_file_formats). It is a sequence of ones and zeros, where different computer architecture have agreed to mean specific instructions for the CPU. Thus an excerpt of the above "Hello World!" executable would look like




```c
0000620 ffff e8ff ff48 ffff 05c6 09e1 0020 5d01
0000630 0fc3 801f 0000 0000 c3f3 0f66 441f 0000
0000640 4855 e589 e95d ff66 ffff 4855 e589 8d48
0000650 9f3d 0000 b800 0000 0000 c1e8 fffe b8ff
0000660 0000 0000 c35d 2e66 1f0f 0084 0000 0000
0000670 5741 5641 8949 41d7 4155 4c54 258d 0736
```





The first column are offsets, and the remaining columns are hexadecimal numbers. For instance 0x55 when seen by the CPU might mean "push rbp". The excerpt above are carefully selected from the executable hexdump because they actually form the instructions for the CPU to print the string "Hello World!" to the standard output! 







In radare2's disassembly of the same executable, we see that 





```c

0x0000064a      55             push rbp
0x0000064b      4889e5         mov rbp, rsp
0x0000064e      488d3d9f0000.  lea rdi, str.Hello_World    ; 0x6f4 ; "Hello World!"
0x00000655      b800000000     mov eax, 0
0x0000065a      e8c1feffff     call sym.imp.printf         ; int printf(const char *format)
0x0000065f      b800000000     mov eax, 0
0x00000664      5d             pop rbp
0x00000665      c3             ret


```




Note that the middle column that starts with 55 and end with c3 are also present in the hexdump lines 0000640 and 0000660. The order in the bytes are flipped due to [little endian ](https://en.wikipedia.org/wiki/Endianness#Little)formatting.







The symbols on the right such as "push rbp" etc are symbolic representations of the binary values. They are called assembly language and serves to provide the human reader an understanding of the instructions.







Next, we look at how the CPU is capable of performing tasks such as arithmetic, or read/write to memory when presented with instructions such as "55 48 89 e5 48 83  ... 5d c3". 







Any [computer algorithm](https://en.wikipedia.org/wiki/Algorithm#Computer_algorithms) can be performed by a [Turing Complete](https://en.wikipedia.org/wiki/Turing_completeness) machine. Almost all modern deterministic computers adheres to the Von Neumann architecture, which in a Turing Complete design.







A typical CPU of the Von Neumann architecture contains the components in the diagram below,  






![](https://upload.wikimedia.org/wikipedia/commons/e/e5/Von_Neumann_Architecture.svg?download)





[Image: Von Neumann architecture](http://By Kapooht - Own work, CC BY-SA 3.0, https://commons.wikimedia.org/w/index.php?curid=25789639)







When the CPU is presented with machine instructions such as "55 48 89 ... 5d c3", it triggers various patterns of electrical signals. Similar to assembly, the electrical signal patterns are based on agreed upon convention.  







In the diagram above, there is a control unit within the CU that contains a program counter that allows the instructions to be operated on sequentially. Each instruction is sent to the arithmetic logic unit (ALU) which then returns certain electrical signals as output. The ALU is made from combinations of logic gates and transistor which enables the deterministic response pattern of  electrical signals. The electrical signals are physical representations of binary values.



