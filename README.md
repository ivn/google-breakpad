# Breakpad for Cygwin/MinGW

google-breakpad with support added for PE/COFF executables with DWARF debugging
information, as used by Cygwin/MinGW

## Compiling

Use ./configure && make (See README)

which will produce dump\_syms, minidump\_dump, minidump\_stackwalk, libbreakpad.a
and for MinGW libcrash\_generation_client.a, libcrash\_generation_server.a, crash\_generation_app.exe

Note that since git-svn ignores svn externals, this repository is missing the
gyp and gtest dependencies.

## Using

See http://code.google.com/p/google-breakpad/wiki/GettingStartedWithBreakpad

### Producing and installing symbols

````
dump_syms crash_generation_app.exe >crash_generation_app.sym
FILE=`head -1 crash_generation_app.sym | cut -f5 -d' '`
BUILDID=`head -1 crash_generation_app.sym | cut -f4 -d' '`
SYMBOLPATH=/symbols/${FILE}/${BUILDID}/
mdir -p ${SYMBOLPATH}
mv crash_generation_app.sym ${SYMBOLPATH}
````

### Generating a minidump file

A small test application demonstrating out-of-process dumping called
crash\_generation\_app.exe is built.

- Run it once, selecting "Server->Start" from the menu
- Run it again, selecting "Client->Deref zero"
- Client should crash, and a .dmp is written to C:\Dumps\

### Processing the minidump to produce a stack trace

````
minidump_stackwalk blah.dmp /symbols/
````

## Issues

### Lack of build-id

Executables produced by Cygwin/MinGW gcc do not currently contain a build-id.
On Windows, this build-id takes the form of a CodeView record.

This build-id is captured for all modules in the process by MiniDumpWriteDump(),
and is used by the breakpad minidump processing tools to find the matching
symbol file.

See http://debuginfo.com/articles/debuginfomatch.html

I have implemented 'ld --build-id' for PE/COFF executables (See
https://sourceware.org/ml/binutils/2014-01/msg00296.html), but you must use
sufficently recent binutils and build with '-Wl,--build-id' to enable that.

A tool could be written to add build-id to existing PE/COFF executables, but in
practice this turns out to be quite tricky...

### Symbols from a PDB or the Microsoft Symbol Server

<a href="http://hg.mozilla.org/users/tmielczarek_mozilla.com/fetch-win32-symbols">
symsrv_convert</a> and dump_syms for PDB cannot be currently built with MinGW,
because they require the MS DIA (Debug Interface Access) SDK (which is only in
paid editions of Visual Studio) and the DIA SDK uses ATL.

An alternate PDB parser is available at https://github.com/luser/dump_syms, but
that also needs some work before it can be built with MinGW.
