Table of Content 
[](https://cs162.org/static/hw/hw-intro/docs/executable/)

# Set up
[Deatils of set up](https://claude.ai/code/artifact/fe196d6b-ee4b-4695-b998-1eb7c8fc7310)

## VS Code

## Docker
Write in Docker
``` 
Help you set up Docker to work with this repository instead AND Clone the workspace repo
git clone https://github.com/Berkeley-CS162/cs162-workspace.git cd cs162-workspace⧉
Build and start it once, in the foreground
Wait for the line Docker workspace is ready!, then stop it with Ctrl+C.

docker-compose up⧉
Start it again, this time in the background
docker-compose up -d⧉
Sanity-check it over SSH
Password is workspace.

ssh workspace@127.0.0.1 -p 16222⧉
Optional: name it so you don't retype the port
Add to ~/.ssh/config:

Host docker162 HostName 127.0.0.1 Port 16222 User workspace⧉
Then it's just ssh docker162 from anywhere, including VS Code.
```
To close Docker
```
docker-compose down
```
To restart it later, just run

```
docker-compose up -d
```
### use VS Code Remote SSH:

Open VS Code
Press Ctrl+Shift+P
Search "Remote-SSH: Connect to Host"
Select docker162
Enter password: workspace

# HW 0
[HW0 Description](https://cs162.org/static/hw/hw-intro/docs/executable/)
## Steps


## Questions
From source code to executable
Now that you’ve seen how map works, let’s take a dive into how we went from high-level C code to an executable.

Before we start, we’ll be using a few compiler flags which are likely new to you. Here’s a summary of the flags we’ll be using.

- Wall – Enables all compiler warnings

- m32 – Compiles the code for the i386 architecture.

- E - Invokes the PREPROCESSOR only.

- S – Invokes the COMPILER only.

- c – Invokes the COMPILER and ASSEMBLER only.

Important: Please use i386-gcc instead of gcc for this homework.

Let’s now invoke the compiler. The compiler takes high-level C code and produces a variant of x86 known as 8086 or i386 assembly.

To compile map.c, run:
```
i386-gcc -m32 -S -o map.S map.c
```
This will only invoke the compiler for map.c and output the assembly code in map.S.

1. Generate recurse.S and find which instructions correspond to the recursive call of recur(i - 1).
```
i386-gcc -m32 -S -o recurse.S recurse.c
```
In recursion.S
```
movl  8(%ebp), %eax   # load i (the parameter) into eax
subl  $1, %eax         # compute i - 1
subl  $12, %esp
pushl %eax             # push (i - 1) as the argument
call  recur             # the recursive call itself
```

Now we will assemble our compiled code into an executable. To assemble our code we can run:
```
i386-gcc -m32 -c map.S -o map.o
```
This turns our raw x86 code (map.S) into machine code or an object file (map.o).

We can also combine these steps by just running i386-gcc -m32 -c on our C file directly. We can run:
```
i386-gcc -m32 -c recurse.c -o recurse.o
```
The assembler converts the raw assembly code into an object file which contains code as well as other data and metadata necessary for execution. Different operating systems use different types of object files. In this class, we will be using ELF (Executable and Linkable Format), the object format used by Linux. Let’s start by taking a look at map.o and recurse.o. These are object files, so we will use the objdump program to read them.
```
i386-objdump -D map.o
i386-objdump -D recurse.o
```

2. What do the .text and .data sections contain? Provide a qualitative description.
The assembler generates a symbol table which is part of the object file. The symbol table contains all the symbols that can be globally referenced (referenced outside the object file) from another object file (i.e. global/static variables and functions).
- .text holds compiled machine instructions
```
Disassembly of section .text:

00000000 <main>:
   0:   8d 4c 24 04             lea    0x4(%esp),%ecx
   4:   83 e4 f0                and    $0xfffffff0,%esp
   7:   ff 71 fc                push   -0x4(%ecx)
   a:   55                      push   %ebp
   b:   89 e5                   mov    %esp,%ebp
   d:   53                      push   %ebx
   e:   51                      push   %ecx
   f:   83 ec 10                sub    $0x10,%esp
  12:   e8 fc ff ff ff          call   13 <main+0x13>
  17:   81 c3 02 00 00 00       add    $0x2,%ebx
  1d:   c7 45 ec 00 00 00 00    movl   $0x0,-0x14(%ebp)
  24:   83 ec 0c                sub    $0xc,%esp
  27:   8d 83 00 00 00 00       lea    0x0(%ebx),%eax
  2d:   50                      push   %eax
  2e:   e8 fc ff ff ff          call   2f <main+0x2f>
  33:   83 c4 10                add    $0x10,%esp
  36:   83 ec 0c                sub    $0xc,%esp
  39:   6a 64                   push   $0x64
  3b:   e8 fc ff ff ff          call   3c <main+0x3c>
  40:   83 c4 10                add    $0x10,%esp
  43:   89 45 f0                mov    %eax,-0x10(%ebp)
  46:   83 ec 0c                sub    $0xc,%esp
  49:   6a 64                   push   $0x64
  4b:   e8 fc ff ff ff          call   4c <main+0x4c>
  50:   83 c4 10                add    $0x10,%esp
  53:   89 45 f4                mov    %eax,-0xc(%ebp)
  56:   83 ec 0c                sub    $0xc,%esp
  59:   6a 03                   push   $0x3
  5b:   e8 fc ff ff ff          call   5c <main+0x5c>
  60:   83 c4 10                add    $0x10,%esp
  63:   b8 00 00 00 00          mov    $0x0,%eax
  68:   8d 65 f8                lea    -0x8(%ebp),%esp
  6b:   59                      pop    %ecx
  6c:   5b                      pop    %ebx
  6d:   5d                      pop    %ebp
  6e:   8d 61 fc                lea    -0x4(%ecx),%esp
  71:   c3                      ret    
```
- .data holds initialized global/static variables
```
Disassembly of section .data:

00000000 <stuff>:
   0:   07                      pop    %es
   1:   00 00                   add    %al,(%eax)
        ...

```
3. What command do we use to view the symbols in an ELF file? (Hint: We can use objdump again, look at man objdump to find the right flag).
-t
```
i386-objdump -t map.o
```
```
map.o:     file format elf32-i386

SYMBOL TABLE:
00000000 l    df *ABS*  00000000 map.c
00000000 l    d  .text  00000000 .text
00000000 l    d  .rodata        00000000 .rodata
00000000 l    d  .text.__x86.get_pc_thunk.bx    00000000.text.__x86.get_pc_thunk.bx
00000000 g     O .bss   00000004 foo
00000000 g     O .data  00000004 stuff
00000000 g     F .text  00000072 main
00000000 g     F .text.__x86.get_pc_thunk.bx    00000000.hidden __x86.get_pc_thunk.bx
00000000         *UND*  00000000 _GLOBAL_OFFSET_TABLE_
00000000         *UND*  00000000 puts
00000000         *UND*  00000000 malloc
00000000         *UND*  00000000 recur
```
Here’s an excerpt from the map.o symbol table:
```
00000000 g O .data 00000004 stuff
00000000 g F .text 00000060 main
...
00000000 *UND* 00000000 malloc
00000000 *UND* 00000000 recur
```
4. What do the g, O, F, and *UND* flags mean?
-  00004008 g O .data ... stuff → g = global (linkable/visible outside this file, vs. local l)
-  O = the symbol is an Object (a variable).
-  00001201 g F .text ... main → F = Function.
-  00000000 F *UND* ... printf@GLIBC_2.0 → *UND* = undefined here: the symbol is used in this file but not defined in it — its real definition is resolved elsewhere (for printf/malloc/puts, that's libc at load time; for recur, it was recurse.o before linking).
<br>

Finally, let’s link our 2 object files to create an executable.
```
i386-gcc -m32 map.o recurse.o -o map
```

Note that we could’ve just called 
```
i386-gcc -m32 map.c recurse.c -o map
```
on the C files to do this entire process in a single command. Often times build systems will separate these commands in order to speed up compile times (since only the changed files need to be recompiled).

```
i386-exec ./map                        # run it
```
```
CS 362 is the best!
i is 3. Address of i is 0x3ffff0b0
i is 2. Address of i is 0x3ffff090
i is 1. Address of i is 0x3ffff070
i is 0. Address of i is 0x3ffff050
```
5. Examine the symbol table of the entire map program now. What has changed? Give a general description, including what happened to recur.
```
i386-objdump -t map
```
```
map:     file format elf32-i386

SYMBOL TABLE:
00000000 l    df *ABS*  00000000              Scrt1.o
000001cc l     O .note.ABI-tag  00000020              __abi_tag
00000000 l    df *ABS*  00000000              crtstuff.c
00003ee8 l     O .ctors 00000000              __CTOR_LIST__
00003ef0 l     O .dtors 00000000              __DTOR_LIST__
000010d0 l     F .text  00000000              deregister_tm_clones
00001110 l     F .text  00000000              register_tm_clones
00001160 l     F .text  00000000              __do_global_dtors_aux
0000400c l     O .bss   00000001              completed.1
00004010 l     O .bss   00000004              dtor_idx.0
000011f0 l     F .text  00000000              frame_dummy
00000000 l    df *ABS*  00000000              crtstuff.c
00003eec l     O .ctors 00000000              __CTOR_END__
00002148 l     O .eh_frame      00000000              __FRAME_END__
000012d0 l     F .text  00000000              __do_global_ctors_aux
00000000 l    df *ABS*  00000000              map.c
00000000 l    df *ABS*  00000000              recurse.c
00000000 l    df *ABS*  00000000              
00003ef8 l     O .dynamic       00000000              _DYNAMIC
0000203c l       .eh_frame_hdr  00000000              __GNU_EH_FRAME_HDR
00003fd0 l     O .got   00000000              _GLOBAL_OFFSET_TABLE_
00000000       F *UND*  00000000              __libc_start_main@GLIBC_2.34
00000000  w      *UND*  00000000              _ITM_deregisterTMCloneTable
000010c0 g     F .text  00000004              .hidden __x86.get_pc_thunk.bx
00004000  w      .data  00000000              data_start
00000000       F *UND*  00000000              printf@GLIBC_2.0
0000400c g       .data  00000000              _edata
0000131c g     F .fini  00000000              .hidden _fini
00001273 g     F .text  00000052              recur
000011f9 g     F .text  00000000              .hidden __x86.get_pc_thunk.dx
00000000  w    F *UND*  00000000              __cxa_finalize@GLIBC_2.1.3
00004008 g     O .data  00000004              stuff
00003ef4 g     O .dtors 00000000              .hidden __DTOR_END__
00000000       F *UND*  00000000              malloc@GLIBC_2.0
00004000 g       .data  00000000              __data_start
00000000       F *UND*  00000000              puts@GLIBC_2.0
00000000  w      *UND*  00000000              __gmon_start__
00004004 g     O .data  00000000              .hidden __dso_handle
00002004 g     O .rodata        00000004              _IO_stdin_used
00004014 g     O .bss   00000004              foo
00004018 g       .bss   00000000              _end
00001090 g     F .text  00000030              _start
00002000 g     O .rodata        00000004              _fp_hw
0000400c g       .bss   00000000              __bss_start
00001201 g     F .text  00000072              main
000012c5 g     F .text  00000000              .hidden __x86.get_pc_thunk.ax
0000400c g     O .data  00000000              .hidden __TMC_END__
00000000  w      *UND*  00000000              _ITM_registerTMCloneTable
000011fd g     F .text  00000000              .hidden __x86.get_pc_thunk.di
00001000 g     F .init  00000000              .hidden _init(#6&#7)
```
- The biggest change is recur: it went from *UND* in map.o alone to 00001273 g F .text 00000052 recur
- because linking in recurse.o gave the linker recur's actual definition to merge in. More generally, the table also grew a lot: it now includes C runtime startup machinery (_start, __libc_start_main, deregister_tm_clones, etc.) that map.o alone didn't have, and addresses are now real load addresses (starting around 0x1000+) instead of starting at 0 like in the unlinked object file. printf, malloc, puts are still *UND* — those stay unresolved until the dynamic linker loads libc.so at runtime.

<br>
objdump can be used to look at more than just the symbol table—it can show us the structure of the executable. Run
```
i386-objdump -x -d map
```

 You will see that your program has several segments, names of functions and variables in your program correspond to labels with addresses or values. The guts of everything is chunks of stuff within segments.

```
map:     file format elf32-i386
map
architecture: i386, flags 0x00000150:
HAS_SYMS, DYNAMIC, D_PAGED
start address 0x00001090

Program Header:
    PHDR off    0x00000034 vaddr 0x00000034 paddr 0x00000034 align 2**2
         filesz 0x00000160 memsz 0x00000160 flags r--
  INTERP off    0x00000194 vaddr 0x00000194 paddr 0x00000194 align 2**0
         filesz 0x00000013 memsz 0x00000013 flags r--
    LOAD off    0x00000000 vaddr 0x00000000 paddr 0x00000000 align 2**12
         filesz 0x00000404 memsz 0x00000404 flags r--
    LOAD off    0x00001000 vaddr 0x00001000 paddr 0x00001000 align 2**12
         filesz 0x00000339 memsz 0x00000339 flags r-x
    LOAD off    0x00002000 vaddr 0x00002000 paddr 0x00002000 align 2**12
         filesz 0x0000014c memsz 0x0000014c flags r--
    LOAD off    0x00002ee8 vaddr 0x00003ee8 paddr 0x00003ee8 align 2**12
         filesz 0x00000124 memsz 0x00000130 flags rw-
 DYNAMIC off    0x00002ef8 vaddr 0x00003ef8 paddr 0x00003ef8 align 2**2
         filesz 0x000000d8 memsz 0x000000d8 flags rw-
    NOTE off    0x000001a8 vaddr 0x000001a8 paddr 0x000001a8 align 2**2
         filesz 0x00000044 memsz 0x00000044 flags r--
EH_FRAME off    0x0000203c vaddr 0x0000203c paddr 0x0000203c align 2**2
         filesz 0x0000003c memsz 0x0000003c flags r--
   STACK off    0x00000000 vaddr 0x00000000 paddr 0x00000000 align 2**4
         filesz 0x00000000 memsz 0x00000000 flags rw-
   RELRO off    0x00002ee8 vaddr 0x00003ee8 paddr 0x00003ee8 align 2**0
         filesz 0x00000118 memsz 0x00000118 flags r--

Dynamic Section:
  NEEDED               libc.so.6
  INIT                 0x00001000
  FINI                 0x0000131c
  GNU_HASH             0x000001ec
  STRTAB               0x000002ac
  SYMTAB               0x0000020c
  STRSZ                0x000000b4
  SYMENT               0x00000010
  DEBUG                0x00000000
  PLTGOT               0x00003fd0
  PLTRELSZ             0x00000020
  PLTREL               0x00000011
  JMPREL               0x000003e4
  REL                  0x000003b4
  RELSZ                0x00000030
  RELENT               0x00000008
  FLAGS                0x00000008
  FLAGS_1              0x08000001
  VERNEED              0x00000374
  VERNEEDNUM           0x00000001
  VERSYM               0x00000360
  RELCOUNT             0x00000002

Version References:
  required from libc.so.6:
    0x09691f73 0x00 04 GLIBC_2.1.3
    0x0d696910 0x00 03 GLIBC_2.0
    0x069691b4 0x00 02 GLIBC_2.34

Sections:
Idx Name          Size      VMA       LMA       File off Algn
  0 .interp       00000013  00000194  00000194  00000194 2**0
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
  1 .note.gnu.build-id 00000024  000001a8  000001a8  000001a8  2**2
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
  2 .note.ABI-tag 00000020  000001cc  000001cc  000001cc 2**2
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
  3 .gnu.hash     00000020  000001ec  000001ec  000001ec 2**2
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
  4 .dynsym       000000a0  0000020c  0000020c  0000020c 2**2
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
  5 .dynstr       000000b4  000002ac  000002ac  000002ac 2**0
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
  6 .gnu.version  00000014  00000360  00000360  00000360 2**1
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
  7 .gnu.version_r 00000040  00000374  00000374  00000374  2**2
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
  8 .rel.dyn      00000030  000003b4  000003b4  000003b4 2**2
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
  9 .rel.plt      00000020  000003e4  000003e4  000003e4 2**2
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
 10 .init         0000002e  00001000  00001000  00001000 2**2
                  CONTENTS, ALLOC, LOAD, READONLY, CODE
 11 .plt          00000050  00001030  00001030  00001030 2**4
                  CONTENTS, ALLOC, LOAD, READONLY, CODE
 12 .plt.got      00000008  00001080  00001080  00001080 2**3
                  CONTENTS, ALLOC, LOAD, READONLY, CODE
 13 .text         00000289  00001090  00001090  00001090 2**4
                  CONTENTS, ALLOC, LOAD, READONLY, CODE
 14 .fini         0000001d  0000131c  0000131c  0000131c 2**2
                  CONTENTS, ALLOC, LOAD, READONLY, CODE
 15 .rodata       00000039  00002000  00002000  00002000 2**2
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
 16 .eh_frame_hdr 0000003c  0000203c  0000203c  0000203c 2**2
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
 17 .eh_frame     000000d4  00002078  00002078  00002078 2**2
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
 18 .ctors        00000008  00003ee8  00003ee8  00002ee8 2**2
                  CONTENTS, ALLOC, LOAD, DATA
 19 .dtors        00000008  00003ef0  00003ef0  00002ef0 2**2
                  CONTENTS, ALLOC, LOAD, DATA
 20 .dynamic      000000d8  00003ef8  00003ef8  00002ef8 2**2
                  CONTENTS, ALLOC, LOAD, DATA
 21 .got          00000030  00003fd0  00003fd0  00002fd0 2**2
                  CONTENTS, ALLOC, LOAD, DATA
 22 .data         0000000c  00004000  00004000  00003000 2**2
                  CONTENTS, ALLOC, LOAD, DATA
 23 .bss          0000000c  0000400c  0000400c  0000300c 2**2
                  ALLOC
 24 .comment      0000002b  00000000  00000000  0000300c 2**0
                  CONTENTS, READONLY
SYMBOL TABLE:
00000000 l    df *ABS*  00000000              Scrt1.o
000001cc l     O .note.ABI-tag  00000020              __abi_tag
00000000 l    df *ABS*  00000000              crtstuff.c
00003ee8 l     O .ctors 00000000              __CTOR_LIST__
00003ef0 l     O .dtors 00000000              __DTOR_LIST__
000010d0 l     F .text  00000000              deregister_tm_clones
00001110 l     F .text  00000000              register_tm_clones
00001160 l     F .text  00000000              __do_global_dtors_aux
0000400c l     O .bss   00000001              completed.1
00004010 l     O .bss   00000004              dtor_idx.0
000011f0 l     F .text  00000000              frame_dummy
00000000 l    df *ABS*  00000000              crtstuff.c
00003eec l     O .ctors 00000000              __CTOR_END__
00002148 l     O .eh_frame      00000000              __FRAME_END__
000012d0 l     F .text  00000000              __do_global_ctors_aux
00000000 l    df *ABS*  00000000              map.c
00000000 l    df *ABS*  00000000              recurse.c
00000000 l    df *ABS*  00000000              
00003ef8 l     O .dynamic       00000000              _DYNAMIC
0000203c l       .eh_frame_hdr  00000000              __GNU_EH_FRAME_HDR
00003fd0 l     O .got   00000000              _GLOBAL_OFFSET_TABLE_
00000000       F *UND*  00000000              __libc_start_main@GLIBC_2.34
00000000  w      *UND*  00000000              _ITM_deregisterTMCloneTable
000010c0 g     F .text  00000004              .hidden __x86.get_pc_thunk.bx
00004000  w      .data  00000000              data_start
00000000       F *UND*  00000000              printf@GLIBC_2.0
0000400c g       .data  00000000              _edata
0000131c g     F .fini  00000000              .hidden _fini
00001273 g     F .text  00000052              recur
000011f9 g     F .text  00000000              .hidden __x86.get_pc_thunk.dx
00000000  w    F *UND*  00000000              __cxa_finalize@GLIBC_2.1.3
00004008 g     O .data  00000004              stuff
00003ef4 g     O .dtors 00000000              .hidden __DTOR_END__
00000000       F *UND*  00000000              malloc@GLIBC_2.0
00004000 g       .data  00000000              __data_start
00000000       F *UND*  00000000              puts@GLIBC_2.0
00000000  w      *UND*  00000000              __gmon_start__
00004004 g     O .data  00000000              .hidden __dso_handle
00002004 g     O .rodata        00000004              _IO_stdin_used
00004014 g     O .bss   00000004              foo
00004018 g       .bss   00000000              _end
00001090 g     F .text  00000030              _start
00002000 g     O .rodata        00000004              _fp_hw
0000400c g       .bss   00000000              __bss_start
00001201 g     F .text  00000072              main
000012c5 g     F .text  00000000              .hidden __x86.get_pc_thunk.ax
0000400c g     O .data  00000000              .hidden __TMC_END__
00000000  w      *UND*  00000000              _ITM_registerTMCloneTable
000011fd g     F .text  00000000              .hidden __x86.get_pc_thunk.di
00001000 g     F .init  00000000              .hidden _init

Disassembly of section .init:

00001000 <_init>:
    1000:       f3 0f 1e fb             endbr32 
    1004:       53                      push   %ebx
    1005:       83 ec 08                sub    $0x8,%esp
    1008:       e8 b3 00 00 00          call   10c0 <__x86.get_pc_thunk.bx>
    100d:       81 c3 c3 2f 00 00       add    $0x2fc3,%ebx
    1013:       8b 83 24 00 00 00       mov    0x24(%ebx),%eax
    1019:       85 c0                   test   %eax,%eax
    101b:       74 02                   je     101f <_init+0x1f>
    101d:       ff d0                   call   *%eax
    101f:       e8 cc 01 00 00          call   11f0 <frame_dummy>
    1024:       e8 a7 02 00 00          call   12d0 <__do_global_ctors_aux>
    1029:       83 c4 08                add    $0x8,%esp
    102c:       5b                      pop    %ebx
    102d:       c3                      ret    

Disassembly of section .plt:

00001030 <__libc_start_main@plt-0x10>:
    1030:       ff b3 04 00 00 00       push   0x4(%ebx)
    1036:       ff a3 08 00 00 00       jmp    *0x8(%ebx)
    103c:       00 00                   add    %al,(%eax)
        ...

00001040 <__libc_start_main@plt>:
    1040:       ff a3 0c 00 00 00       jmp    *0xc(%ebx)
    1046:       68 00 00 00 00          push   $0x0
    104b:       e9 e0 ff ff ff          jmp    1030 <_init+0x30>

00001050 <printf@plt>:
    1050:       ff a3 10 00 00 00       jmp    *0x10(%ebx)
    1056:       68 08 00 00 00          push   $0x8
    105b:       e9 d0 ff ff ff          jmp    1030 <_init+0x30>

00001060 <malloc@plt>:
    1060:       ff a3 14 00 00 00       jmp    *0x14(%ebx)
    1066:       68 10 00 00 00          push   $0x10
    106b:       e9 c0 ff ff ff          jmp    1030 <_init+0x30>

00001070 <puts@plt>:
    1070:       ff a3 18 00 00 00       jmp    *0x18(%ebx)
    1076:       68 18 00 00 00          push   $0x18
    107b:       e9 b0 ff ff ff          jmp    1030 <_init+0x30>

Disassembly of section .plt.got:

00001080 <__cxa_finalize@plt>:
    1080:       ff a3 20 00 00 00       jmp    *0x20(%ebx)
    1086:       66 90                   xchg   %ax,%ax

Disassembly of section .text:

00001090 <_start>:
    1090:       f3 0f 1e fb             endbr32 
    1094:       31 ed                   xor    %ebp,%ebp
    1096:       5e                      pop    %esi
    1097:       89 e1                   mov    %esp,%ecx
    1099:       83 e4 f0                and    $0xfffffff0,%esp
    109c:       50                      push   %eax
    109d:       54                      push   %esp
    109e:       52                      push   %edx
    109f:       e8 18 00 00 00          call   10bc <_start+0x2c>
    10a4:       81 c3 2c 2f 00 00       add    $0x2f2c,%ebx
    10aa:       6a 00                   push   $0x0
    10ac:       6a 00                   push   $0x0
    10ae:       51                      push   %ecx
    10af:       56                      push   %esi
    10b0:       ff b3 28 00 00 00       push   0x28(%ebx)
    10b6:       e8 85 ff ff ff          call   1040 <__libc_start_main@plt>
    10bb:       f4                      hlt    
    10bc:       8b 1c 24                mov    (%esp),%ebx
    10bf:       c3                      ret    

000010c0 <__x86.get_pc_thunk.bx>:
    10c0:       8b 1c 24                mov    (%esp),%ebx
    10c3:       c3                      ret    
    10c4:       66 90                   xchg   %ax,%ax
    10c6:       66 90                   xchg   %ax,%ax
    10c8:       66 90                   xchg   %ax,%ax
    10ca:       66 90                   xchg   %ax,%ax
    10cc:       66 90                   xchg   %ax,%ax
    10ce:       66 90                   xchg   %ax,%ax

000010d0 <deregister_tm_clones>:
    10d0:       e8 24 01 00 00          call   11f9 <__x86.get_pc_thunk.dx>
    10d5:       81 c2 fb 2e 00 00       add    $0x2efb,%edx
    10db:       8d 8a 3c 00 00 00       lea    0x3c(%edx),%ecx
    10e1:       8d 82 3c 00 00 00       lea    0x3c(%edx),%eax
    10e7:       39 c8                   cmp    %ecx,%eax
    10e9:       74 1d                   je     1108 <deregister_tm_clones+0x38>
    10eb:       8b 82 1c 00 00 00       mov    0x1c(%edx),%eax
    10f1:       85 c0                   test   %eax,%eax
    10f3:       74 13                   je     1108 <deregister_tm_clones+0x38>
    10f5:       55                      push   %ebp
    10f6:       89 e5                   mov    %esp,%ebp
    10f8:       83 ec 14                sub    $0x14,%esp
    10fb:       51                      push   %ecx
    10fc:       ff d0                   call   *%eax
    10fe:       83 c4 10                add    $0x10,%esp
    1101:       c9                      leave  
    1102:       c3                      ret    
    1103:       8d 74 26 00             lea    0x0(%esi,%eiz,1),%esi
    1107:       90                      nop
    1108:       c3                      ret    
    1109:       8d b4 26 00 00 00 00    lea    0x0(%esi,%eiz,1),%esi

00001110 <register_tm_clones>:
    1110:       e8 e4 00 00 00          call   11f9 <__x86.get_pc_thunk.dx>
    1115:       81 c2 bb 2e 00 00       add    $0x2ebb,%edx
    111b:       55                      push   %ebp
    111c:       89 e5                   mov    %esp,%ebp
    111e:       53                      push   %ebx
    111f:       8d 8a 3c 00 00 00       lea    0x3c(%edx),%ecx
    1125:       8d 82 3c 00 00 00       lea    0x3c(%edx),%eax
    112b:       83 ec 04                sub    $0x4,%esp
    112e:       29 c8                   sub    %ecx,%eax
    1130:       89 c3                   mov    %eax,%ebx
    1132:       c1 e8 1f                shr    $0x1f,%eax
    1135:       c1 fb 02                sar    $0x2,%ebx
    1138:       01 d8                   add    %ebx,%eax
    113a:       d1 f8                   sar    %eax
    113c:       74 14                   je     1152 <register_tm_clones+0x42>
    113e:       8b 92 2c 00 00 00       mov    0x2c(%edx),%edx
    1144:       85 d2                   test   %edx,%edx
    1146:       74 0a                   je     1152 <register_tm_clones+0x42>
    1148:       83 ec 08                sub    $0x8,%esp
    114b:       50                      push   %eax
    114c:       51                      push   %ecx
    114d:       ff d2                   call   *%edx
    114f:       83 c4 10                add    $0x10,%esp
    1152:       8b 5d fc                mov    -0x4(%ebp),%ebx
    1155:       c9                      leave  
    1156:       c3                      ret    
    1157:       8d b4 26 00 00 00 00    lea    0x0(%esi,%eiz,1),%esi
    115e:       66 90                   xchg   %ax,%ax

00001160 <__do_global_dtors_aux>:
    1160:       f3 0f 1e fb             endbr32 
    1164:       55                      push   %ebp
    1165:       89 e5                   mov    %esp,%ebp
    1167:       57                      push   %edi
    1168:       e8 90 00 00 00          call   11fd <__x86.get_pc_thunk.di>
    116d:       81 c7 63 2e 00 00       add    $0x2e63,%edi
    1173:       56                      push   %esi
    1174:       53                      push   %ebx
    1175:       83 ec 0c                sub    $0xc,%esp
    1178:       80 bf 3c 00 00 00 00    cmpb   $0x0,0x3c(%edi)
    117f:       75 61                   jne    11e2 <__do_global_dtors_aux+0x82>
    1181:       8b 87 20 00 00 00       mov    0x20(%edi),%eax
    1187:       85 c0                   test   %eax,%eax
    1189:       74 13                   je     119e <__do_global_dtors_aux+0x3e>
    118b:       83 ec 0c                sub    $0xc,%esp
    118e:       ff b7 34 00 00 00       push   0x34(%edi)
    1194:       89 fb                   mov    %edi,%ebx
    1196:       e8 e5 fe ff ff          call   1080 <__cxa_finalize@plt>
    119b:       83 c4 10                add    $0x10,%esp
    119e:       8d b7 20 ff ff ff       lea    -0xe0(%edi),%esi
    11a4:       8d 9f 24 ff ff ff       lea    -0xdc(%edi),%ebx
    11aa:       8b 87 40 00 00 00       mov    0x40(%edi),%eax
    11b0:       29 f3                   sub    %esi,%ebx
    11b2:       c1 fb 02                sar    $0x2,%ebx
    11b5:       83 eb 01                sub    $0x1,%ebx
    11b8:       39 d8                   cmp    %ebx,%eax
    11ba:       73 1a                   jae    11d6 <__do_global_dtors_aux+0x76>
    11bc:       8d 74 26 00             lea    0x0(%esi,%eiz,1),%esi
    11c0:       83 c0 01                add    $0x1,%eax
    11c3:       89 87 40 00 00 00       mov    %eax,0x40(%edi)
    11c9:       ff 14 86                call   *(%esi,%eax,4)
    11cc:       8b 87 40 00 00 00       mov    0x40(%edi),%eax
    11d2:       39 d8                   cmp    %ebx,%eax
    11d4:       72 ea                   jb     11c0 <__do_global_dtors_aux+0x60>
    11d6:       e8 f5 fe ff ff          call   10d0 <deregister_tm_clones>
    11db:       c6 87 3c 00 00 00 01    movb   $0x1,0x3c(%edi)
    11e2:       8d 65 f4                lea    -0xc(%ebp),%esp
    11e5:       5b                      pop    %ebx
    11e6:       5e                      pop    %esi
    11e7:       5f                      pop    %edi
    11e8:       5d                      pop    %ebp
    11e9:       c3                      ret    
    11ea:       8d b6 00 00 00 00       lea    0x0(%esi),%esi

000011f0 <frame_dummy>:
    11f0:       f3 0f 1e fb             endbr32 
    11f4:       e9 17 ff ff ff          jmp    1110 <register_tm_clones>

000011f9 <__x86.get_pc_thunk.dx>:
    11f9:       8b 14 24                mov    (%esp),%edx
    11fc:       c3                      ret    

000011fd <__x86.get_pc_thunk.di>:
    11fd:       8b 3c 24                mov    (%esp),%edi
    1200:       c3                      ret    

00001201 <main>:
    1201:       8d 4c 24 04             lea    0x4(%esp),%ecx
    1205:       83 e4 f0                and    $0xfffffff0,%esp
    1208:       ff 71 fc                push   -0x4(%ecx)
    120b:       55                      push   %ebp
    120c:       89 e5                   mov    %esp,%ebp
    120e:       53                      push   %ebx
    120f:       51                      push   %ecx
    1210:       83 ec 10                sub    $0x10,%esp
    1213:       e8 a8 fe ff ff          call   10c0 <__x86.get_pc_thunk.bx>
    1218:       81 c3 b8 2d 00 00       add    $0x2db8,%ebx
    121e:       c7 45 ec 00 00 00 00    movl   $0x0,-0x14(%ebp)
    1225:       83 ec 0c                sub    $0xc,%esp
    1228:       8d 83 38 e0 ff ff       lea    -0x1fc8(%ebx),%eax
    122e:       50                      push   %eax
    122f:       e8 3c fe ff ff          call   1070 <puts@plt>
    1234:       83 c4 10                add    $0x10,%esp
    1237:       83 ec 0c                sub    $0xc,%esp
    123a:       6a 64                   push   $0x64
    123c:       e8 1f fe ff ff          call   1060 <malloc@plt>
    1241:       83 c4 10                add    $0x10,%esp
    1244:       89 45 f0                mov    %eax,-0x10(%ebp)
    1247:       83 ec 0c                sub    $0xc,%esp
    124a:       6a 64                   push   $0x64
    124c:       e8 0f fe ff ff          call   1060 <malloc@plt>
    1251:       83 c4 10                add    $0x10,%esp
    1254:       89 45 f4                mov    %eax,-0xc(%ebp)
    1257:       83 ec 0c                sub    $0xc,%esp
    125a:       6a 03                   push   $0x3
    125c:       e8 12 00 00 00          call   1273 <recur>
    1261:       83 c4 10                add    $0x10,%esp
    1264:       b8 00 00 00 00          mov    $0x0,%eax
    1269:       8d 65 f8                lea    -0x8(%ebp),%esp
    126c:       59                      pop    %ecx
    126d:       5b                      pop    %ebx
    126e:       5d                      pop    %ebp
    126f:       8d 61 fc                lea    -0x4(%ecx),%esp
    1272:       c3                      ret    

00001273 <recur>:
    1273:       55                      push   %ebp
    1274:       89 e5                   mov    %esp,%ebp
    1276:       53                      push   %ebx
    1277:       83 ec 04                sub    $0x4,%esp
    127a:       e8 46 00 00 00          call   12c5 <__x86.get_pc_thunk.ax>
    127f:       05 51 2d 00 00          add    $0x2d51,%eax
    1284:       8b 55 08                mov    0x8(%ebp),%edx
    1287:       83 ec 04                sub    $0x4,%esp
    128a:       8d 4d 08                lea    0x8(%ebp),%ecx
    128d:       51                      push   %ecx
    128e:       52                      push   %edx
    128f:       8d 90 4c e0 ff ff       lea    -0x1fb4(%eax),%edx
    1295:       52                      push   %edx
    1296:       89 c3                   mov    %eax,%ebx
    1298:       e8 b3 fd ff ff          call   1050 <printf@plt>
    129d:       83 c4 10                add    $0x10,%esp
    12a0:       8b 45 08                mov    0x8(%ebp),%eax
    12a3:       85 c0                   test   %eax,%eax
    12a5:       7e 14                   jle    12bb <recur+0x48>
    12a7:       8b 45 08                mov    0x8(%ebp),%eax
    12aa:       83 e8 01                sub    $0x1,%eax
    12ad:       83 ec 0c                sub    $0xc,%esp
    12b0:       50                      push   %eax
    12b1:       e8 bd ff ff ff          call   1273 <recur>
    12b6:       83 c4 10                add    $0x10,%esp
    12b9:       eb 05                   jmp    12c0 <recur+0x4d>
    12bb:       b8 00 00 00 00          mov    $0x0,%eax
    12c0:       8b 5d fc                mov    -0x4(%ebp),%ebx
    12c3:       c9                      leave  
    12c4:       c3                      ret    

000012c5 <__x86.get_pc_thunk.ax>:
    12c5:       8b 04 24                mov    (%esp),%eax
    12c8:       c3                      ret    
    12c9:       66 90                   xchg   %ax,%ax
    12cb:       66 90                   xchg   %ax,%ax
    12cd:       66 90                   xchg   %ax,%ax
    12cf:       90                      nop

000012d0 <__do_global_ctors_aux>:
    12d0:       f3 0f 1e fb             endbr32 
    12d4:       e8 20 ff ff ff          call   11f9 <__x86.get_pc_thunk.dx>
    12d9:       81 c2 f7 2c 00 00       add    $0x2cf7,%edx
    12df:       8b 82 18 ff ff ff       mov    -0xe8(%edx),%eax
    12e5:       83 f8 ff                cmp    $0xffffffff,%eax
    12e8:       74 2e                   je     1318 <__do_global_ctors_aux+0x48>
    12ea:       55                      push   %ebp
    12eb:       89 e5                   mov    %esp,%ebp
    12ed:       53                      push   %ebx
    12ee:       8d 9a 18 ff ff ff       lea    -0xe8(%edx),%ebx
    12f4:       83 ec 04                sub    $0x4,%esp
    12f7:       8d b4 26 00 00 00 00    lea    0x0(%esi,%eiz,1),%esi
    12fe:       66 90                   xchg   %ax,%ax
    1300:       ff d0                   call   *%eax
    1302:       8b 43 fc                mov    -0x4(%ebx),%eax
    1305:       83 eb 04                sub    $0x4,%ebx
    1308:       83 f8 ff                cmp    $0xffffffff,%eax
    130b:       75 f3                   jne    1300 <__do_global_ctors_aux+0x30>
    130d:       8b 5d fc                mov    -0x4(%ebp),%ebx
    1310:       c9                      leave  
    1311:       c3                      ret    
    1312:       8d b6 00 00 00 00       lea    0x0(%esi),%esi
    1318:       c3                      ret    

Disassembly of section .fini:

0000131c <_fini>:
    131c:       f3 0f 1e fb             endbr32 
    1320:       53                      push   %ebx
    1321:       83 ec 08                sub    $0x8,%esp
    1324:       e8 97 fd ff ff          call   10c0 <__x86.get_pc_thunk.bx>
    1329:       81 c3 a7 2c 00 00       add    $0x2ca7,%ebx
    132f:       e8 2c fe ff ff          call   1160 <__do_global_dtors_aux>
    1334:       83 c4 08                add    $0x8,%esp
    1337:       5b                      pop    %ebx
    1338:       c3                      ret    (#11)
```

In the objdump output these segments are under the section heading. There’s actually a slight nuance between these two terms which you can read more about online.
<br>
Using the output of objdump, answer the following questions:

6. What segment(s)/section(s) contains recur (the function)? (The address of recur in objdump will not be exactly the same as what you saw in gdb. An optional stretch exercise is to think about why. Hint: See the Wikipedia article on relocation.)
- see 00001273 g F .text 00000052 recur in the symbol table, and 00001273 <recur>: under "Disassembly of section .text" in the same output.
- 
7. What segment(s)/section(s) contains global variables? Hint: look for the variables foo and stuff.
- 00004008 g O .data 00000004 stuff: initialized: volatile int stuff = 7
- 00004014 g O .bss 00000004 foo: declared but never given a value: int foo;

8. Do you see the stack or heap segment anywhere? Explain.
- No
- However, in the Program Header filesz 0x00000000 memsz 0x00000000 flags rw-, its file size and memory size are both 0.
- here's no heap entry anywhere. no real stack or heap content appears in the file, because neither exists until the OS actually starts running the process.

9. Based on the output of map, in which direction does the stack grow? (Reminder: Please use
```
i386-exec ./map
```
to run map.)
```
CS 362 is the best!
i is 3. Address of i is 0x3ffff0b0
i is 2. Address of i is 0x3ffff090
i is 1. Address of i is 0x3ffff070
i is 0. Address of i is 0x3ffff050
```
Each deeper recursive call gets a smaller address, so the stack grows downward, toward lower memory addresses.
<br>
When you ran map, you might have noticed that it prints "CS362 is the best!". However, we wanted to print "CS162 is the best!".

Let’s see what happened by invoking the preprocessing stage. The compiler takes your C code and will output new C code. What does this really do? Time to find out!

To preprocess map.c, run:
```
i386-gcc -m32 -E -o map.i map.c
```
10. You can see that gcc produces a map.i that is far larger than the original map.c file. Notice that define directives perform string replacement.
- map.i will exist and be much bigger than map.c with all #includes and macros expanded inline.

11. Modify Makefile to make sure that "CS162 is the best!" is printed instead. You may not modify or add any other files. Hint: Refer to this page from the GCC documentation.
```
map: map.c
	$(CC) $(CFLAGS) -DCS162 map.c recurse.c -o map
```

# HW 1: List
[](https://cs162.org/static/hw/hw-list/)

## Getting started
To get started, log in to your development environment and get the starter code.
```
cd ~/code/personal/
git pull staff main
cd hw-list
```
To build the code, run make, which should create four binaries: pthread, words, pwords, and lwords. Make sure not to commit and push any binaries when submititng to the autograder. You can get rid of unnecessary binaries using make clean.

Note: You do not have to add the -f flag to run the executable in this assignment.

### Skeleton
You’ll notice that for some files, only the object files without the source are provided. Part of being able to program means being able to work with the abstractions you’re provided, so we’ve left out implementations which are not necessary for you to complete this assignment.s

- list.h provides the Pintos list abstraction which is taken directly from the Pintos source code. list.c provides the implementations, but you should be able to use this library solely based on the API given in list.h. You must not modify these files.

- word_count.h defines the API you will implement. We have already provided necessary data structures word_count_t and word_count_list_t which you must use.

- words.o and word_count.o provide compiled implementations of methods necessary to run the words program from Homework Intro. You can use the outputs of these programs as sanity checks on what your other programs that you’ll build should output.

- word_count_l.c will house your implementation of the the API in word_count.h using Pintos lists. The Makefile will provide the macro definition of PINTOS_LIST when compiling. When word_count_l.c is linked with the driver in lwords.o and compiled, it should result in an application lwords that behaves identically to the frequency mode of words but internally using Pintos lists instead of traditional linked lists as seen in Homework Intro.

- Similarly, word_count_p.c will house your implementation of the API in word count.h using Pintos lists and proper synchronization of the word count data structure for a multithreaded program. Unlike lwords, you’ll need to write the driver program in pwords.c. When word_count_p.c and pwords.c are put together and compiled, it should create an application pwords that behaves identically as lwords and frequency mode of words but internally uses multiple threads.

- word_helpers.h provides an API for parsing and counting words. word_helpers.o provides compiled implementations for these methods.

- pthread.c implements an example application that creates multiple threads andprints out certain memory addresses and values. You may find it helpful to base your pwords.c implementation off of pthread.c. While you won’t be writing any code in pthread.c, you will be reading and analyzing it.

## Steps1: lwords
First, read list.h to understand the API. Focus on the examples given in the big docstring in the beginning of the file.

Next, thoroughly read through the data structures and methods in word_count.h. In particular, pay attention to the word_count_t and word_count_list_t structs. You may find it beneficial to see the compiler flags used in the Makefile and how that affects the struct definitions.

Finally, complete word_count_l.c to properly implement the word count API given in word_count.h. You must use the Pintos list API. After you finish making this change, lwords should work properly (i.e. exhibit the same behavior as frequency mode of words).

The wordcount_sort function sorts the wordcount list according to the comparator passed as an argument. Although lwords uses the less count function from word_helpers.h as the less argument, the wordcount sort function should be generic enough to work with any valid comparator passed in as the less argument. For example, passing the less word function from word_helpers.h as the less parameter should. Check out some basics on function pointers if you’re having trouble understanding and writing the syntax.\


## Steps2: pthread
Read pthread.c carefully. Then, run make and run pthread multiple times and observe its output. Answer the following questions based on your observations.

1. Is the program’s output the same each time it is run? Why or why not?

2. Based on the program’s output, do multiple threads share the same stack?

3. Based on the program’s output, do multiple threads share the same global variables?

4. Based on the program’s output, what is the value of void *threadid? How does this relate to the variable’s type (void *)?

5. Using the first command line argument, create a large number of threads in pthread. Do all threads run before the program exits? Why or why not? Note: Please be precise. Vague responses will not be given credit.

## Steps3: pwords
Table of contents
Implementation
Synchronization
Gradescope questions

### Implementation
words and lwords operate in a single thread, opening, reading, and processing each file one after another. With pwords, your task is to provide the same end-to-end functionality while using multiple threads. This means you are not allowed to materialize your intermediate results (i.e. write each thread’s results to a separate file and aggregate these). Make sure to read through word_count.h and Makefile to see what macros are used for pwords. In particular, the word_count_list_t will differ from lwords.

pwords.c will serve as the driver program, meaning it will manage the creation and upkeep of the threads you create. Each file should be processed in a separate thread. We recommend you reference pthread.c to draw inspiration and clues as to how you should structure your code.

### Synchronization
When implementing words_count_p.c, keep in mind the functionality of all these methods are identical to what you did in words_count_l.c. However, you must use synchronization techiques to ensure coordination amongst different threads to prevent race conditions.

Your synchronization must be fine-grained. Different threads should be able to open and read their respective files concurrently, serializing only their modifications to shared data. In particular, it is unacceptable to use a global lock around the call to the count_words function in pwords.c, since it would prevent multiple threads from concurrently reading files. We reserve the right to deduct points from such implementations. Instead, you should only synchronize access to the word count list data structure in word_count_p.c. You will need to ensure all such modifications are complete before printing the result or terminating the process.

We recommend that you start by just implementing the thread-per-file aspect (i.e. without synchronization). This will be quite similar to the code you’ve written in word_count_l.c. Your program might not even error, since multithreaded programs with synchronization bugs may appear to work properly much of the time. Once you’re confident in the logic of your methods, then add in the necessary synchronization.

To help you find subtle synchronization bugs in your program, we have provided a decently large input for your words program in the gutenberg/ directory. These files were generated from select stories from Project Gutenberg, making sure to choose short stories so that the word count program does not take too long to run. Make sure to compare the results of running pwords to running words to check if your output is correct. As stated before, this does not ensure your synchronization is correct, but it might alert you to subtle synchronization bugs that may not manifest for smaller inputs.

### Gradescope questions
After correctly implementing pwords (i.e. passing autograder tests), compare lwords and pwords by answering the following questions.

Briefly compare the performance of lwords and pwords when run on the Gutenberg dataset. How might you explain their relative performance? Simply note and explain what you observe, whether or not the result is what you expected! Hint: Look into man time.

Under what circumstances would pwords perform better than lwords? Under what circumstances would lwords perform better than pwords? Is it possible to use multithreading in a way that always performs better than lwords?




## Questions
