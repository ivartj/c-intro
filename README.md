I write this in English because the topic is most often written about
in English, so that it will be easier to recognize terms in other
resources. I will attempt to write this more opiniated and brief than
what you will find elsewhere so to set you in a direction quickly.

I should note that I generally avoid Integrated Development
Environments in favour of the command line, and have little knowledge
with how to use Visual Studio.

GENERAL INFORMATION
-------------------

The C programming language is designed to be a low-level and
performant programming language that is easily adapted to a variety of
computer architectures (such as x86, x86-64, PowerPC, ARM and
SPARC). It is used in the foundations of most modern software, such as
operating systems and Java runtimes. It lacks many modern innovations
such as a built-in object system and garbage collection.

The lack of garbage collection means that you manually allocate and
free additional memory used by the process through functions. If a
process continually allocates memory without freeing unused memory, it
is known as a "memory leak." It is also possible for programs written
in C to attempt to access memory that the operating system has not
granted it access to, which will cause the process to crash with a
"segfault". Things like these make programming in C more error prone
than programming in other languages.

COMPILERS
---------

There are many C compilers. The following are the four that I think
will be of most interest:

* TCC, Tiny C Compiler, is a small compiler that is available in a
  binary distribution for Windows. It is very easy to install and try
  out C in. It is not much used for serious programming projects.
  
  http://bellard.org/tcc/

* GCC, GNU Compiler Collection, is one of the most full-featured C
  compilers and is the one that is most used on Linux and FreeBSD. It is
  available as MinGW on Windows, which also provides a Unix-like
  command-line environment.

  http://gcc.gnu.org/

  http://mingw.org/

* Visual Studio and Windows SDK includes a C compiler. It is often
  criticized for not complying with modern standards. C codebases will
  often have to make adjustments to be able to be compiled in Visual
  Studio. Microsoft's support for C++ is much better.

* Clang, based on LLVM, is the compiler used most on Mac OS X and is
  increasingly displacing GCC in many environments.

BASIC COMPILING ENVIRONMENT
---------------------------

To use the compiler, you need to have the directory of the executable in your PATH environmental variable, so that you don't have the type out the full path all the time. This directory is typically called "bin" for binary.

On Unix systems, you can inspect and append to the variable like this:

    echo $PATH
    export PATH=$PATH:/path/to/executable/directory

On Windows

    echo %PATH%
    PATH=%PATH%;C:\Path\to\Executable\Directory\

The environmental variable is reset once you open a new prompt. You can avoid redoing this by writing a script, but I will not cover that unless you request it.

BASIC COMPILING
---------------

Given that you are in a directory with a `main.c` source file, and the name of the executable file for your compiler is `cc`, type

   cc main.c

and it should produce an `a.out` executable file on Unix, or maybe `a.exe` when on Windows [ ? ]. On Unix systems this file can be executed with

    ./a.out

On Windows the file needs the `.exe` extension to be executed, and you can then type

    a.exe

The output filename can be changed with the `-o` option. Like so

    cc -o prog.exe main.c

which should produce a `prog.exe` file.

When compiling multiple source files which collectively only have one main function, you can either add more source files to the previous command, or break down the compiling into multiple steps, like so

    cc -c main.c
    cc -c lib.c
    cc -o prog main.o lib.o

(The lib.c file typically has a corresponding `lib.h` header file which is `#include`ed in `main.c`)

This is often done by tools that automate the compilation steps so to reduce the time needed to recompile the code after minor changes. If `lib.c` is changed, then step 2 and 3 need to be redone, but not step 1.

MAKE
----

`make` is a family of command-line programs that help you automate compilation. Typically you write a file called `Makefile`, and when you then run `make` (or `nmake` if you use the Windows SDK) in that directory it does everything needed to (re)compile the codebase. The Makefile for the previous 3 steps could be written as

    prog: main.o lib.o
    	$(CC) -o prog main.o lib.o

    main.o: main.c
    	$(CC) -c main.c

    lib.o: lib.c
    	$(CC) -c lib.c

Each of these three directives tell that to make this file (`prog`) I need these files (`main.o`, `lib.o`) and will produce it by doing the following (`$(CC) -o prog main.o lib.o`).

$(CC) is here a variable that should point to the default C compiler of the system. If it doesn't work you can run make like this

    make CC=cc

where `cc` is the compiler you want to use.
