---
title: Notes for better_tutorial in Nasm
tags:
  - Assembly
  - NASM
categories:
  - Notes
mathjax: true
sitemap: false
date: 2019-05-20 15:02:24
---

This note is based on the tutorial of [this project](https://github.com/he-mat/better_tutorial)

# Chapter 1 & 2

##  [NASM Data Type](https://nekosecurity.com/x86-64-assembly/part-4-data-types)

| Type | Meaning            | Size     |
| ---- | ------------------ | -------- |
| db   | Define byte        | 1 byte   |
| dw   | Define word        | 2 bytes  |
| dd   | Define double word | 4 bytes  |
| dq   | Define quad word   | 8 bytes  |
| dt   | Define ten bytes   | 10 bytes |

<!--more-->

## [*global* directive](https://stackoverflow.com/a/17899048/6742691)

- NASM specific  
- entry point in executable file  

- [default entry point name](https://nasm.us/doc/nasmdoc7.html#section-7.4.6): _start 

- to change the entry point name, specify -e parameter

  `ld -e my_entry_point -o out a.o` 



## [int 80h & syscall](https://stackoverflow.com/a/12806910/6742691)

- syscall is invalid on 32-bit , only on x86-64

- `int 0x80` is a legacy way to invoke a system call and should be avoided.

- vSDO: [1](http://adam8157.info/blog/2011/10/linux-vdso/) [2](https://cloud.tencent.com/developer/article/1073909)

- [sysenter](http://articles.manugarg.com/systemcallinlinux2_6.html)

  

## [CR/LF](https://knowledge.ni.com/KnowledgeArticleDetails?id=kA00Z0000019KZDSA2)

- `0Ah` is **L**ine **F**eed character, equals` \n`.  將游標向下移，但不回到句首

- `0Dh` is **C**arriage **R**eturn character, equals `\r`. 將游標移到該行的句首



# Chapter 3

## CMP

`cmp destination, source`   其實就等於 `destination - source`

CMP instruction would change Zero, Carry, Overflow, Auxiliary and Parity flags.



Unsigned integer operation

| CMP Results          | ZF   | CF   |
| -------------------- | ---- | ---- |
| Destination < Source | 0    | 1    |
| Destination > Source | 0    | 0    |
| Destination = Source | 1    | 0    |

Signed integer operation

| CMP Results          | Flags   |
| -------------------- | ------- |
| Destination < Source | SF = OF |
| Destination > Source | SF ≠ OF |
| Destination = Source | ZF = 1  |



## JZ

`jz <label>`  jump to label if ZF = 1



## INC

`inc operand`   operand + 1 byte



## [EFLAGS Register](http://finalfrank.pixnet.net/blog/post/22992166-x86-cpu-暫存器-register-大全)

### Zero Flag

The flag is set when the result of an operation generates a result of zero

### Carry Flag

- **Unsigned** arithmetic operation
- The flag is set in two situations
  - 兩者相加的結果（大小）大於Register能存放的空間
  - 當小數減掉大數時。
    - 00000001 - 00000010 = 11111111  and CF is set (表示負號，因為是Unsigned)

### Overflow Flag

與Carry Flag相同，但用於**Signed** arithmetic operation

### [Auxiliary Flag](https://en.wikipedia.org/wiki/Adjust_flag)

### Sign Flag

when the result of an operation generates a negative result

### Parity Flag

以`11101101`做例子，因為有6個1，是偶數個，所以PF=1



# Chapter 5

`%include "<filename.inc>"`    Include other source files into current code，類似C的include.   [Reference](https://nasm.us/doc/nasmdoc4.html#section-4.6.1)



# Chapter 6

`sys_write` 以 `0h` (NULL byte) 作為string終止符號(而非`0Ah` or `0Dh`)。所以如果想要`sys_write`在data段某個string，但是若每一個string最後都沒有以`0h`做結尾，那將會印出從給予位置到第一個null byte才會結束。



# Chapter 9

## Section .bss

- **B**lock **S**tarted by **S**ymbol

- 用於存放尚未初始化的變數。該段通常只包含變數名以及其長度(大小)
- e.g.

```
SECTION .bss
variableName1:      resb    1  ;reserve one byte
variableName2:      resw    1  ;reserve one word
variableName3:      resd    1  ;reserve one double word
variableName4:      resq    1  ;reserve one quad word
variableName5:      rest    1  ;reserve one extended precision float
```



## sys_read

- `eax`  3 
- `ebx` 0 :  read from file descriptor STDIN
- `ecx` :  address of the reserved space
- `edx` :  number of  bytes to read



# Chapter 11

## div (unsigned division)

$dest ← dest \div source$

| Dividend | Divisor | Quotient | Remainder |
| -------- | ------- | -------- | --------- |
| AX       | r/m8    | AL       | AH        |
| DX:AX    | r/m16   | AX       | DX        |
| EDX:EAX  | r/m32   | EAX      | EDX       |
| RDX:RAX  | r/m64   | RAX      | RDX       |

Example

```
SECTION .data
dividend        dq      0000000800300020h
divisor         dd      00000100h

SECTION .text
  global _start

_start:
  mov   edx, [dividend+4]
  mov   eax, [dividend]
  div   dword [divisor]	; need to specify the size of the content by using memory address
```

- **idiv (Signed divide)**



# Chapter 12

`add dest, source` 

   $dest ← dest + source$

- EFLAGS are set according to the result

- Check [this table](https://www.nasm.us/xdoc/2.09.07/html/nasmdocb.html#section-B.1.2) to find available combination of dest and source



# Chapter 13

`sub dest, source`   

  $dest ← dest - source$



# Chapter 14

`mul src`  

 $eax ← eax \times src$



# Chapter 16

## Initial Process Stack

當主程式開始前，OS(exec)會先初始化stack，使其包含一些重要的資訊，如下：

|                         High Address                         |
| :----------------------------------------------------------: |
|                      other system stuff                      |
|                             ...                              |
| [Auxiliary Vectors](http://articles.manugarg.com/aboutelfauxiliaryvectors.html)    *4 words each* |
|                       Zero doubleword                        |
|             Address of last environment argument             |
|                             ...                              |
|              Address of environment argument 2               |
|       Address of environment argument 1   *doubleword*       |
|                       Zero doubleword                        |
|                   Address of Last Argument                   |
|                             ...                              |
|                    Address of Argument 2                     |
|             Address of Argument 1   *doubleword*             |
|           Argument Count [**esp**]   *doubleword*            |
|                         Low Address                          |

**esp的位址**一開始會存放argument的總數。而接下來存放的都是各argument的位址。Argument1預設是放置該程式的路徑，所以user輸入的參數會是從Argument2開始。

在tutorial中，利用pop可以依序得到user輸入的command line argument。

Reference [1](http://refspecs.linuxfoundation.org/ELF/zSeries/lzsabi0_zSeries/x895.html)  [2](https://www.dreamincode.net/forums/topic/285550-nasm-linux-getting-command-line-parameters/)



## Set register to zero

`xor reg, reg`  該instruction是比較建議的方式來將register歸零，而非`mov reg, 0`

```
xor     reg          
     10110110
     10110110
   ------------
     00000000
```

 [Refer](https://stackoverflow.com/a/33668295/6742691)



## 在gdb中輸入command line argument

`gdb --args  executable_name   arg1  arg2  arg3`

[Refer](https://stackoverflow.com/questions/6121094/how-do-i-run-a-program-with-commandline-arguments-using-gdb-within-a-bash-script)



# Chapter 17

## local labels

透過namespace使相同名字的label能夠重複使用，如下

```
label1:  
    .loop:  
       ...  
       jmp .loop			;這會跳到label1底下的.loop而非label2底下的.loop
  
label2:
    .loop:  
       ...  
       jmp .loop  
  
jmp label1.loop		;當從其他地方要呼叫時，只要將global label(namespace)配上local label即可access
```

[Reference](https://www.tortall.net/projects/yasm/manual/html/nasm-local-label.html)



# Chapter 19

## sys_execve

*executes a new program.*

`eax`  11

`ebx`  address of the file 

`ecx`  address of arguments

`edx`  address of the environment variables

the address points to the variable in the section .data

（其實該函數可以討論的更深，但我覺得之後研究OS再深入做筆記）



# Chapter 20

## sys_fork

*invoke a child process*

`eax`  2

After invoking sys_fork, if `eax`  is 0, then it implies that the current process is a child process.
