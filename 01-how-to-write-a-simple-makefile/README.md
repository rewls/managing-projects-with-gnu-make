# Chapter 1. How to Write a Simple Makefile

- The `make` program is intended to automate the mundane aspects of transforming source code into an executable.

- The advantages of `make` over scripts is that you can specify the relationships between the elements of your program to `make`, and it knows through these relationships and timestamps exactly what steps need to be redone to produce the desired program each time.

- `make` defines a language for describing the relationships between source code, intermediate files, and executables.

- It also provides features to manage alternate configurations, implement reusable libraries of specifications, and parameterize processes with user-defined macros.

- The specification that `make` uses is generally saved in a file named *makefile*.

- Here is a *makefile* to build the traditional "Hello, World" program:

```makefile
hello: hello.c
	gcc hello.c -o hello
```

- To build the program execute `make` by typing at the command prompt of your favorite shell:

```sh
$ make
gcc hello.c -o hello
```

- This will cause the `make` program to read the *makefile* and build the first target it finds there.

- If a target is included as a command-line argument, that target is updated.

- If no command-line targets are given, then the first target in the file is used, called the *default goal*.

- Typically the default goal in most *makefiles* is to build a program.

- This usually involves many steps.

- Often the source code for the program is incomplete and the source must be generated using utilities such as `flex` or `bison`.

- Next the source is compiled into binary object files.

- Then for C/C++, the object files are bound together by a linker to form an executable program.

- Modifying any of the source files and reinvoking `make` will cause some, but usually not all, of these commands to be repeated so the source code changes are properly incorporated into the executable.

- The specification file, or *makefile*, describes the relationship between the source, intermediate, and executable program files so that `make` can perform the minimum amount of work necessary to update the executable.

- `make` is flexible enough to be used anywhere one kind of file depends on another from traditional programming in C/C++ to Java, TeX, database management, and more.

## Targets and Prerequisites

- Essentially a *makefile* contains a set of rules used to build an application.

- The first rule seen by `make` is used as the *default rule*.

- A *rule* consists of three parts: the target, its prerequisites, and the command(s) to perform:

```makefile
*target*: *prereq1* *prereq2*
	*commands*
```

- The *target* is the file or thing that must be made.

- The *prerequisites* or *dependents* are those files that must exist before the target can be successfully created.

- And the *commands* are those shell commands that will create the target from the prerequisites.

- Here is a rule for compiling a c file, *foo.c*, into an object file, *foo.o*:

```makefile
foo.o: foo.c foo.h
	gcc -c foo.c
```

- The command script usually appears on the following lines and is preceded by a tab character.

- when `make` is asked to evaluate a rule, it begins by finding the files indicated by the prerequisites and target.

- If any of the prerequisites has an associated rule, `make` attempts to update those first.

- If any prerequisite is newer than the target, the target is remade by executing the commands.

- If any of the commands generates an error,, the building of the target is terminated and `make` exits.

- One file is considered newer than another if it has been modified more recently.

- Here is a program to count the number of occurrences of the words "fee," "fie," "foe," and "fum" in its input.

- It uses a `flex` scanner driven by a simple main:

> [count_words.c](count_words.c)

> [lexer.l](lexer.l)

> [Makefile](Makefile)

```sh
$ make
gcc -c count_words.c
flex -t lexer.l > lexer.c
gcc -c lexer.c
gcc count_words.o lexer.o -lfl -o count_words
```

- The order in which commands are executed by `make` are nearly the opposite to the order they occur in the *makefile*.

- This *top-down* style is common in *makefiles*.

- The `make` program supports this style in many ways.

- Chief among these is `make`'s two-phase execution model and recursive variables.

- We will discuss these in great detail in later chapters.

## Dependency Checking

- The `-l` option to `gcc` indicates a system library that must be linked into the application.

- The actual library name indicated by "fl" is *libfl.a*.

- GNU `make` includes special support for this syntax.

- When a prerequisite of the form-`l<NAME>` is seen, `make` searches for a file of the form *libNAME.so*; if no match is found, it then searches for *libNAME.a*.

- Here `make` finds */usr/lib/libfl.a*.

    - In my environment, `make` finds */lib/libfl.so*.

## Minimizing Rebuilds

- The problem is that we have forgotten some rules in our lexical analyzer and `flex` is passing this unrecognized text to its output.

- To solve this problem we simply add an "any character" rule and a newline rule:

```
.
\n
```

- After editing this file we need to rebuild the application to test our fix.

```sh
$ make
flex -t lexer.l > lexer.c
gcc -c lexer.c
gcc count_words.o lexer.o -lfl -o count_words
```

- Notice this time the file *count_words.c* was not recompiled.

## Invoking make

- The previous examples assumes that:

    - All the project source code and the `make` description file are stored in a single directory.

    - The `make` description file is called *makefile*, *Makefile*, or *GNUMakefile*.

    - The *makefile* resides in the user's current directory when executing the `make` command.

- One of the most useful `make` command-line option is `--just-print` (or `-n`) which tells `make` to display the commands it would execute for a particular target without actually executing them.

- It is also possible to set almost any *makefile* variable on the command line to override the default value or the value set in the *makefile*.

## Basic Makefile Syntax

- *Makefile* are usually structured top-down so that the most general target, often called `all`, is updated by default.

- More and more detailed targets follow with targets for program maintenance, such as a `clean` target to delete unwanted temporary files, coming last.

- Targets do not have to be actual files, any name will do.

- The more complete form of a rule is:

```c
*target1* *target2* *target3*: *prerequisite1* *prerequisite2*
	*command1*
	*command2*
	*command3*
```

- One or more targets appear to the left of the colon and zero or more prerequisites can appear to the right of the colon.

- The set of commands executed to update a target are sometimes called the *command script*, but most often just the *commands*.

- Each command *must* begin with a tab character.

- This syntax tells `make` that the characters that follow the tab are to be passed to a subshell for execution.

- We'll discuss the complexities of the tab character in Chapter 2.

- The comment character for `make` is the hash or pound sign, `#`.

- All text from the pound sign to the end of line is ignored.

- Comments can be indented and leading whitespace is ignored.

- The comment character `#` does not introduce a `make` comment in the text of commands.

- Long lines can be continued using the standard Unix escape character backslash.

- Later we'll cover other ways of handling long prerequisite lists.
