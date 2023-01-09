---
title: Notes for NASM Learning
tags:
  - NASM
  - Assembly
categories:
  - Notes
mathjax: true
sitemap: false
date: 2019-04-30 18:22:24
---

# Command

## MacOS

### 64-bit
```
> nasm -f macho64 <filename.asm> -o <object_filename.o>  
> ld -macosx_version_min 10.7.0 -lSystem -o <executable> <object_filename.o>  
> ./<executable>  
```

### 32-bit
```
> /usr/local/bin/nasm -f macho <filename.asm> -o <object_filename.o>  
> ld -macosx_version_min 10.7.0 -o <executable> <object_filename.o>   
> ./<executable>  
```

<!--more-->

## Linux

### 64-bit
```
> nasm -f elf64 <filename.asm>  -o <object_filename.o>  
> ld -o <executable> <object_filename.o>  
> ./<executable>   
```

### 32-bit
```
> nasm -f elf <filename.asm>  -o <object_filename.o>  
> ld -m elf_i386 -o <executable> <object_filename.o>  
> ./<executable>   
```
- 加上 `-m elf_i386`，set emulation to 32-bit。在64-bit上的電腦compile 32-bit的檔案時，需加上這行。




# System Call Table
- [Reference for MacOS](https://opensource.apple.com/source/xnu/xnu-1504.3.12/bsd/kern/syscalls.master)
	- first column is system call number  
- [System Call Table for linux 32-bit](https://www.informatik.htw-dresden.de/~beck/ASM/syscall_list.html)
- [System Call Table for linux 64-bit](https://blog.rchapman.org/posts/Linux_System_Call_Table_for_x86_64/)
- `find . -name 'syscall.h'`
- 在mac 64-bit時，呼叫syscall時，需要[`rax = Code + 0x2000000`](https://stackoverflow.com/a/53905561/6742691) 
	- e.g.  sys_write => `0x2000004`

***Notes!!***  在32-bit MacOS上呼叫syscall與linux不同。MacOS 是將參數丟到stack上然後呼叫syscall，而非丟到register，所以無法直接將Linux 32-bit nasm code拿到MacOS上。[解釋點我](https://filippo.io/making-system-calls-from-assembly-in-mac-os-x/)


# objdump

## Options

- `-i`  列出可支援`-b`的檔案格式和`-m`的架構
- `-f` 顯示File header資訊。 e.g. File format, architecture, flags and start address
- `-h` 顯示Section Header。 e.g.  `.text` `.data`
- `-t` 顯示[Symbol Table]([http://wen00072.github.io/blog/2014/12/09/field-descriptions/](http://wen00072.github.io/blog/2014/12/09/field-descriptions/))
- `-p` 顯示Program Header Table
- `-x`  combination of `-f` `-h` `-t` `-p`
- `-d` 反組譯 code section
- `-D` 反組譯 ALL section
- `-b` 反組譯指定的[BFD格式](https://en.wikipedia.org/wiki/Binary_File_Descriptor_library)
- `-m` 反組譯用的架構

# readelf
- 與objdump的區別
  - 不提供反組譯
  - 不借助BFD，直接讀取ELF file → **訊息比較多**

## Options

- `-a` equivalent to: `-h` `-l` `-S`  `-s`  `-r` ` -d` ` -V` ` -A` ` -I`


# addr2line

Translate an address in an executable or an offset in a section of an object file into file names and line numbers

nasm在編譯時，必須加上`-g`  (to generate debug information)。`addr2line` only supports functions that have debug information

### Options

- `-a [addr, addr2..]` 
- `-e filename` the name of the executable
- `-p` pretty print


