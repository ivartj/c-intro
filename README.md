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
operating systems and programming language implementations. It lacks
many modern innovations such as a built-in object system and garbage
collection.

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

* Visual Studio and Windows SDK includes a C compiler called `cl`. It is often
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

and it should produce an `a.out` executable file on Unix, or maybe `a.exe` when on Windows. On Unix systems this file can be executed with

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

Each of these three directives tell that to make this file (`prog`) I
need these files (`main.o`, `lib.o`) and will produce it by doing the
following (`$(CC) -o prog main.o lib.o`).

`$(CC)` is here a variable that should point to the default C compiler
of the system. If it doesn't work you can run make like this

    make CC=cc

where `cc` is the compiler you want to use.

There is more Makefile syntax not covered here, but it is often not
present in all `make` implementations. Other tools that automate
compilation, such as Autotools and CMake, work by writing the Makefile
for you. I will not cover either of these as their usage can be quite
complicated, and `make` suffice for smaller projects.

USING LIBRARIES
---------------

There are a great many libraries available to C, as major libraries are frequently written in C and have C as its primary interface. This includes

 * SDL, Simple DirectMedia Layer, which is a cross-platform library
   that provides an audio and window environment, among other
   things. Used in many games.

 * OpenGL, an API for hardware accelerated computer graphics. Among
   the major implementations is Mesa 3D.

 * SQLite, a widely used embedded database management system. It is
   used to store information in Firefox and Google Chrome.

In addition to installing and `#include`ing the header files of these libraries, you also need to tell the compiler things like in what directory it will find the header files (.h files) and the library files (typically named with a `lib` prefix and with extensions .a, .la, .so , .dll or .dylib).

This is done through compiler flags, but they are generally not given manually. It does help to be familiar with the following however:

`-I` tells the compiler an additional directory to search for header files. This is used when producing object files (.o files).

The following two options is used when "linking", when producing an executable:

`-L` tells the compiler an additional directory to search for library files.

`-l` tells the compiler to link to a library in one of the library directories. For instance, to link to libSDL.la, that would be

    -lSDL

### PKG-CONFIG

In Unix-style development environments, these flags are typically
provided by a tool called `pkg-config`. For instance, on my Mac OS X
system, I can do this:

    $ pkg-config --cflags --libs sdl
    -D_GNU_SOURCE=1 -D_THREAD_SAFE -I/opt/local/include/SDL -L/opt/local/lib -lSDLmain -Wl,-framework,AppKit -lSDL -Wl,-framework,Cocoa 

`pkg-config`, or `pkgconf` which is an alternative I use on Windows,
searches through .pc files installed with the libraries in which these
flags are given.

In a GNU Makefile the output of `pkg-config` can be used like this:

    CFLAGS=$(shell pkg-config --cflags sdl gl glu sqlite3)
    LDFLAGS=$(shell pkg-config --libs sdl gl glu sqlite3)
    
    prog: main.o lib.o
    	$(CC) -o prog main.o lib.o $(LDFLAGS)

    main.o: main.c
    	$(CC) $(CFLAGS) main.c
    
    lib.o: lib.c
    	$(CC) $(CFLAGS) lib.c

### SHARED LIBRARIES / DYNAMIC-LINK LIBRARIES (DLLs)

If you attempt to run a program that uses a shared library, you may
encounter an error unless the library is in the environment's search paths for shared libraries. Statically linked libraries do not have this problem as all their necessary components are included in the executable upon linking.

Here are the basics of search paths on major systems:

Windows searches the directory of the executable and the paths in the `PATH` environmental variable for DLL files.

Linux systems searches standard directories for shared libraries, such as `/usr/lib`, and the paths in the `LD_LIBRARY_PATH` environmental variable for .so files.

Mac OS X searches the directory of the executable as well as the `DYLD_LIBRARY_PATH` environmental variable for .dylib files.