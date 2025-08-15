# zlib for AmigaOS 4

Source: [https://zlib.net/](https://zlib.net/)

This makefile can be used to compile the above library and create a new release lha archive, to be used with the AmigaOS 4 SDK, for native and cross compiling.

**Supported libc**
- newlib
- clib2
- clib4

## How to compile
```
make init
make build
```

The clib that will be used while compiling can be selected by using the `LIBC=` argument.
Valid options are newlib, clib2 and clib4
```
make build LIBC=clib4
```

To create a release package the following steps need to be done.
```
make init
make release
```

## Bugs and todo
- You can dynamically link to zlib with clib2 and clib4, but you would need the correct libc.so and libgcc.so in the same directory as the program for it to load the correct shared libraries. Otherwise the program will crash.
