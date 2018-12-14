# Make / Overview

**This documentation will be expanded as examples are created.**

The `make` program is used to control how software is compiled.
It is an older technology that was developed for C and other languages, and is still an effective tool.
The basic usage is to create a file named `makefile` and then run the `make` command,
which will detect and use the `makefile`.
The `makefile` includes instructions for controlling a workflow,
typically to compile software, but it can be used to perform other tasks.

See the following resources:

* [Make (software) on Wikipedia](https://en.wikipedia.org/wiki/Make_(software))
* [make man page](https://linux.die.net/man/1/make)
* [Mrbook's Tutorial on Makefiles](http://mrbook.org/blog/tutorials/make/)
* [TutorialsPoint Tutorial](https://www.tutorialspoint.com/makefile/)

Sections on this page:

* [Technical Topics](#technical-topics)
	+ [Printing Usage](#printing-usage)
	+ [Setting Variable to Program Output](#setting-variable-to-program-output)
	+ [Customizing Makefile for Operating System](#customizing-makefile-for-operating-system)
	+ [Customizing Makefile for Compiler](#customizing-makefile-for-compiler)

--------------------------

## Technical Topics ##

The following sections provide information about specific technical topics that are useful.

### Printing Usage ###

It is helpful to print the makefile targets so that it is clear what can be done by the makefile.
It may be appropriate to make the help the default target so that a different target must be specified.
This can avoid confusion by requiring a target rather than just running with `make`.

For example, add a target as the default.
See the [source file in GitHub](https://github.com/OpenCDSS/cdss-app-statemod-fortran/blob/master/src/main/fortran/makefile).

```
default: help
```

Then create a target that prints the usage when `make` or `make help` are run:

```
# help: print the targets that are available and useful configuration information
help:
        @echo "--------------------------------------------------------------------------------"
        @echo "StateMod makefile targets:"
        @echo ""
        @echo "all        Default target that prints this help message."
        @echo "clean      Remove dynamically created files (but not final executable)."
        @echo "help       Print this message."
        @echo "statemod   Compile the StateMod executable, recompiling any .o if .for modified."
        @echo "veryclean  Make the 'clean' target, and also remove the final executable."
        @echo ""
        @echo "file.o     Compile the source file.for file into object file file.o,"
        @echo "           useful to check syntax for a single file."
        @echo "--------------------------------------------------------------------------------"
        @echo "Important makefile variables that are used:"
        @echo ""
        @echo "FC (compiler) = $(FC)"
        @echo "STATEMOD_VERSION (from statem.for) = $(STATEMOD_VERSION)"
        @echo "--------------------------------------------------------------------------------"
        @echo "Important environment variables that are used:"
        @echo ""
        @echo "OS (to determine operating system) = $(OS)"
        @echo "--------------------------------------------------------------------------------"
```

### Setting Variable to Program Output ###

It is often useful to set a variable to the output of a program.
One example is to detect the version of a program being compiled so the version can be used in output files,
such as the executable program name.
The important syntax is to use the `:= $(shell ...)` syntax to assign the standard output from
a program (or sequence of programs) to a variable.
See the [source file in GitHub](https://github.com/OpenCDSS/cdss-app-statemod-fortran/blob/master/src/main/fortran/makefile).

```
STATEMOD_VERSION := $(shell cat statem.for | grep 'ver =' | grep -v 'xx' | cut -d '=' -f 2 | sed "s/'//g" | tr -d ' ' )
```

### Customizing Makefile for Operating System ###

It is helpful to ensure that a makefile is portable across operating systems,
or at least for the target operating systems.
This allows a single makefile to be used and maintained.
The operating system may be a clear distinction (e.g., Linux and Windows) or a more nuanced distinction
(e.g., Cygwin and Git Bash, which are Linux-like but run on top of Windows).

Detecting the operating system on Linux could be done by using the `uname` program,
for example search `uname` output for `CYGWIN` and if found assume that the operating system is Cygwin.
However, this will not work if the operating system on which `make` is being run does not have a `uname` program.
Therefore, it may be necessary to try a hierarchy of checks to determine the operating system.

The Windows operating system defines the environment variable `OS` that indicates the operating system, for example:

```
C:\>echo %OS%
Windows_NT
```

The value of this environment variable may not be important to the `make` command but the existence of the variable
is a good indicator that the operating system is Windows.
Therefore, logic like the following can be used in the makefile, in this case to specify different source
files for each operating system, in this case subroutines that deal with paths (slashes, colons, etc.).
See the [source file in GitHub](https://github.com/OpenCDSS/cdss-app-statemod-fortran/blob/master/src/main/fortran/makefile).

```
ifdef OS
        # Assume Windows legacy naming convention
        getpath_o_file = getpath.o
        putpath_o_file = putpath.o
else
        # Assume Linux
        getpath_o_file = getpath_linux.o
        putpath_o_file = putpath_linux.o
endif
```

Alternate approaches can be taken to achieve the same result.
The above file list would then be included in the dependency list as follows:

```
some-program: \
        statem.o \
                namext.o \
                $(parse_o_file) \
                $(getpath_o_file) \
```

Similarly, the operating system check can be used to control the output executable name:

```
ifdef OS
        # Windows...
        $(FC) $(FFLAGS) -o statemod-$(STATEMOD_VERSION)-gfortran-32bit.exe $^ $(LDFLAGS)
        @echo "-----------------------------------------------------------------------"
        @echo "Executable is statemod-$(STATEMOD_VERSION)-gfortran-32bit.exe"
        @echo "-----------------------------------------------------------------------"
else
        # Linux...
        $(FC) $(FFLAGS) -o statemod-$(STATEMOD_VERSION)-gfortran-32bit $^ $(LDFLAGS)
        @echo "-----------------------------------------------------------------------"
        @echo "Executable is statemod-$(STATEMOD_VERSION)-gfortran-32bit"
        @echo "-----------------------------------------------------------------------"
endif
```

## Customizing Makefile for Compiler ##

Similar to how a makefile can be [customized for the operating system](#customizing-makefile-for-operating-system),
it is possible to customize the makefile for a compiler.
The `ifeq` syntax shown below compares the value of the `FC` variable to the literal string `gfortran` and
if equal the instructions in first clause are used.
The following example illustrates how the compiler name controls files that should be included.
See the [source file in GitHub](https://github.com/OpenCDSS/cdss-app-statemod-fortran/blob/master/src/main/fortran/makefile).

```
# Compiler-specific routines to handle command line parsing and date/time processing
ifeq ($(FC),gfortran)
        # gfortran code version
        parse_o_file = parse_gfortran.o
        dattim_o_file = dattim_gfortran.o
else
        # Legacy Lahey code version
        parse_o_file = parse.o
        dattim_o_file = dattim.o
endif
```
