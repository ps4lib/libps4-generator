libps4-generator
=====

> Used to generate libps4 through libps4-boilerplate, FreeBSD (libps4-std-headers) and PS4 module (libps4-sce-headers) headers.

## Description
The generator is used to rebuild the libps4 library to extend its functionality. If you are just interested in using the library, please see libps4.

It imports FreeBSD C and POSIX headers as well as PS4 Module headers into include/. It analyzes them and extracts all function declarations it can find (using eliben/pycparser).

The symbols (names) of those function declarations are then verified to exist on the PS4 (see resolver/). When all detected symbols (currently ~1300) have been checked, a stub is created for those which where detected (see libps4-boilerplate for more).

The generator creates different stubs for syscalls, functions and modules. Each stub is generated as one translation unit (file) to reduce the net result of linked binary. Once the generator has finished, libps4 can be build using make.

The boilerplate and the generator are intended to provide a self contained toolset to generate the libps4 library. The functionality should eventually stabilize as the main attention shifts towards the development (RE) of the headers and modules.

The internals of the generated files are explained in more detail in libps4-boilerplate.

## Install

```
* install python version 3
* install pycparser (e.g. pip install)
* to enable checking: adapt the remote address in generate.conf and build and run resolve on the ps4
```

##Use
```
python3 generate.py <command> <args>

clean [source|include]          # imported files
import <std|sce|all>            # FreeBSD or (SCE) module headers
collect                         # symbols in imported headers
[check]                         # symbols on the PS4 (requires resolver and code exec)
[fake]                          # Poor mans check - mostly correct - some functions may not work
boil                            # copies boilerplate to source/ and include/
generate                        # source/ from include/
default                         # perform configured actions (see config)
```

##Internals, Hints and Warnings
###Import
FreeBSD and Module headers will be maintained for your convenience in libps4-std-headers and libps4-sce-headers.

To get FreeBSD includes manually, download an ISO (e.g. 9.0) and extract or mount it. You can try >9.0 (e.g 9.3) ISOs to gain access to certain C11 headers, missing under 9.0.

As the ISO may contain symbolic links in include/ - feedback or fixes for Windows platforms are welcome.

###Check and Fake
To check for the existence of symbols, you need to execute code. This is not yet public knowledge.

Fake is not intended as a stable basis to produce code. Certain functions will not perform correctly (NOP and return -1) though most will work out of the box. All module functions will work. All syscalls will work. Most C functions will work (pthreads for example will not).

###Conventions
The library and generator have a preference for convention over configuration when it comes to the way code is generated and headers are named. For example: When a module is named `libSceLibcInternal.sprx` the corresponding header is expected to be named `libc_internal.h` and vice versa. This ensures that different branches can be used as drop in replacements. You can see a list of expected headers in sce.h. Should the library be named out of convention (e.g. libkernel), sce.h (a pseudo C header) provides an 'as' mechanism to indicate the correct name. The header includes a list of all currently known modules.

##Files

###Riles of the repo
```
generator.py|.conf              # Generation (gluing) + config
std.h                           # C and POSIX headers in one large pseudo header
sce.h                           # PS4 (SCE) module headers in one large pseudo header
								# Both parsed and recursively imported
```

###Content created in libps4
```
libps4/include/*                # Headers which are imported (std and/or sce)
libps4/source/*                 # Stubs generated from headers and found on the PS4
libps4/include|source/internal  # base boilerplate, copied on boil
include/sys/syscall.h           # base syscall map overrides on boil
Makefile, crt0.s, make          # base boilerplate, copied on boil
```

###File generated and provided up to date for your convenience in ps4dev/libps4-symbols
```
symbols.json                # Generated by 'collect' used by 'generate'
							# Amended by 'fake' and 'check'
```

##TODO
- Windows compatibility and checking
- Python2 support?
- Add support for non-generated extensions (implementations) to libps4, aside from libps4-boilerplate
- Enable different targets e.g. syscall, std, sce only, fast (via direct fw based offsets)
- Cleanup code (python uber-skilled rewrite and changes are welcome - just for the kick of it)
- Wait for bugs, help and feature requests (all welcome)
- Rewrite in C <(^^<)?
