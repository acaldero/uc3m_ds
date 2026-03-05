
# Summary of the main aspects of using Make for Distributed Systems
+ **Felix Garcia Carballeira and Alejandro Calderon Mateos**
+ [![License: CC BY-NC-SA 4.0](https://img.shields.io/badge/License-CC%20BY--NC--SA%204.0-blue.svg)](https://github.com/acaldero/uc3m_ds/blob/main/LICENSE)


## Contents

* Getting started:
  * [Requirements](#requirements)
  * [Sample code for compiling with Makefile](#sample-code-for-compiling-with-makefile)
* From manual compilation to using Makefile with rules:
   * [Compilation WITHOUT Makefile](#compilation-without-makefile-manual-compilation)
   * [Compilation with simple Makefile](#compilation-with-makefile-first-version)
   * [Compilation with Makefile with variables](#compilation-with-makefile-second-version-with-variables)
   * [Compilation with Makefile with rules](#compilation-with-makefile-third-version-with-rules)
   * [Compilation with Makefile with special rules](#compilation-with-makefile-fourth-version-with-special-rules)
* [Additional materials](#recommended-materials)


## Requirements

The main requirements are:
* An Internet connection to consult documentation.
* Access to a Linux machine with development software installed:
   * You can use the Virtual Classrooms of the Computer Science Department Laboratory:
     * ["https://www.lab.inf.uc3m.es/servicios/aulas-virtuales-del-laboratorio/](https://www.lab.inf.uc3m.es/servicios/aulas-virtuales-del-laboratorio/)
   * You can use your Debian/Ubuntu with: ```sudo apt-get install build-essential gdb ddd```


## Example code to compile with Makefile

We will use three files:

* lib_hola.h
  ```c
  void di_hola ( void ) ;
  ```

* lib_hello.c
  ```c
  void say_hello ( void )
  {
     printf("Hello World...") ;
  }
  ```

* main.c
  ```c
  #include "lib_hello.h"

  int main ( int argc, char *argv[] )
  {
      say_hello() ;
      return 0 ;
  }
  ```


## Compilation WITHOUT Makefile (manual compilation)

To compile the above files, execute the following commands:

```bash
gcc -g -Wall -c lib_hola.c -o lib_hola.o
gcc -g -Wall -c main.c     -o main.o
gcc -g -Wall -o main main.o lib_hola.o
```


## Compilation with Makefile (first version)

We will initially use the following file:

* Makefile:
  ```bash
  all:
		gcc -g -Wall -c lib_hola.c -o lib_hola.o
		gcc -g -Wall -c main.c     -o main.o
		gcc -g -Wall -o main main.o lib_hola.o
  ```

To use the Makefile file, you must execute:
```bash
make
```

### Basic structure of the Makefile file

The skeleton of the previous Makefile is:
```bash
targets: prerequisites
	command 1
	command 2
	command 3
```
To obtain a target, two elements are indicated:
  
* Prerequisites: these are targets that must be met first
  * Commands (command 1/2/3): once the prerequisites are met, the way to achieve the target is to execute the commands one after the other
    > Important: do not use spaces before each "gcc -g ...", **only tabs**.


## Compiling with Makefile (second version with variables)

We will use the following makefile file:

* Makefile:
  ```bash
  CC=gcc
  CFLAGS=-g -Wall

  all:
		$(CC) $(CFLAGS) -c lib_hola.c -o lib_hola.o
		$(CC) $(CFLAGS) -c main.c     -o main.o
		$(CC) $(CFLAGS) -o main main.o lib_hola.o
  ```

To use the Makefile file, run:
```bash
make
```


### Variables in the Makefile file

To define a variable, you can use:
```bash
VARIABLE=value
```

To use the value of the variable, use:
```bash
$(VARIABLE)
```

> Important: The variable name cannot contain spaces.


## Compiling with Makefile (third version with rules)

We will use the following makefile file:

* Makefile:
  ```bash
  CC=gcc
  CFLAGS=-g -Wall
  OBJS=main.o lib_hola.o

  all: $(OBJS)
  $(CC) $(CFLAGS) -o main $(OBJS)
  
  lib_hola.o: lib_hola.c
      $(CC) $(CFLAGS) -c lib_hola.c -o lib_hola.o

  main.o: main.c
      $(CC) $(CFLAGS) -c main.c     -o main.o
	  
  clean: $(OBJS)
      rm -fr $(OBJS)
  ```

To use the Makefile file, it is recommended to run:
```bash
make clean; make
```


## Compilation with Makefile (fourth version with special rules)

* Makefile file:
  ```bash
  CC=gcc
  CFLAGS=-g -Wall
  OBJS=main.o lib_hola.o

  .PHONY: all clean

  all: $(OBJS)
  $(CC) $(CFLAGS) -o main $(OBJS)
  
  %.o: %.c
      @echo "Compiling... $<"
      $(CC) $(CFLAGS) -c $< -o $@

  clean: $(OBJS)
      @echo "Removing $(OBJS)..."
      rm -fr $(OBJS)
  ```

To use the Makefile file, it is recommended to run:
```bash
make clean; make
```

There are two special rules:
* The rule to ignore files called all and clean (otherwise, doing a make all if there is an all file assumes that the rule is fulfilled and is not executed):
  ```bash
  .PHONY: all clean
  ```
* Use a single rule to indicate that each .o file (%.o) requires an associated .c file (%.c) and is compiled with ```-c $< -o $@``` where ```$<``` is the name of the .c file and ```$@``` is the name of the .o file:
  ```bash
  %.o: %.c
	  	@echo “Compiling... $<”
		$(CC) $(CFLAGS) -c $< -o $@
  ```


## Recommended material
* https://makefiletutorial.com/
* https://www3.nd.edu/~zxu2/acms60212-40212/Makefile.pdf

