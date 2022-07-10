# A BASIC GUIDE TO SHARED LIBRARIES IN C & C#

## CREATE A SHARED OBJECT FILE WITH CLANG (C)

---

A shared object file (e.g., mylib.dll or mylib.so) is compiled code that you can access & use during your program's execution.

We now want to compile the C source code below into a shared object file.

```c
/* mylib.c */
#include <stdio.h>

extern void print_hello(void)
{
  puts("Hello World!");
}
```

To compile & link to a (dynamically linkable) shared object file in Linux, do

`clang -fpic -shared mylib.c -o mylib.so`

Alternatively, you can also compile and then link, doing

`clang -c -fpic mylib.c`

and

`clang -shared mylib.o -o mylib.so`

```
The compiler flag `-fpic` makes your code position independent or relocatable.
Thus, "fpic" stands for "position independent code".
Programs or processes only know their own addresses.
So, if you don't include `-fpic`, your library function `print_hello()` is out of reach within another process.
Another way to grasp `-fpic` is to think of it as a "placeholder mechanism" for memory addresses.

Your operating system's program loader sets the final addresses for you via the "placeholder addresses" from `-fpic`.

The `-shared` flag tells your compiler to produce a shared object file (e.g., mylib.dll or mylib.so).
```

Now we have a shared library file named `mylib.so`.

## USE C FUNCTIONS FROM A SHARED OBJECT IN C

You can access the `print_hello` function from `mylib.so` within another C program.

For this, you need these Linux operating system functions

- `dlopen` (loads shared object file)
- `dlsym` (returns function addresses)
- `dlclose` (unloads shared object)

You also need to include the `dlfcn.h` header and use the above Linux operating system functions like so

```c
/* myapp.c */
#include <stddef.h>
#include <stdio.h>

#include <dlfcn.h>

int main()
{
    typedef void (* Function_ptr)(void);
    void* handle = NULL;
    Function_ptr ptr_to_print_hello = NULL;

    handle = dlopen("./mylib.so", RTLD_LAZY);

    ptr_to_print_hello = (Function_ptr)dlsym(handle, "print_hello");

    ptr_to_print_hello(); /* Hello World! */

    dlclose(handle);
}
```

Compile & link that via

`clang myapp.c -ldl -o app`

```
To tell the linker to look for the shared object file, we use the `-ldl` flag.
```

Then run it. We (should) get

`Hello World!`

## USE C FUNCTIONS FROM A SHARED OBJECT IN C#

You can access the `print_hello` function from `mylib.so` in C#.

For this, you need to tell C# how to deal with C functions (i.e., `print_hello`). We have also a word for that, it is "marshalling".

To marshall C's `print_hello` (from `mylib.so`) for C#, do

```cs
// myapp.cs
using System;
using System.Runtime.InteropServices;

internal static class FFI
{
  [DllImport("mylib.so", CallingConvention = CallingConvention.Cdecl)]
  internal static extern void print_hello();
}

public class TestFFI
{
  public static void Main(string[] args)
  {
    FFI.print_hello();
  }
}
```

Then compile it with Mono, doing

`mcs myapp.cs`

Finally, run it via

`mono myapp.exe`

You should get

```
Hello World!
```

## WORDS

---

1. **Handle:** _You typically use an handle to indirectly refer to a programming resource (e.g., a global object) managed by the implementation.
   Pointers (and references), on the other hand, enable you to use the resources (pointees) directly. Example: `int fd = open(...);` Where `fd` is a file handle managed by the operating system._
2. **Foreign function interface (FFI):** _A feature which makes it possible to call & use functions from one programming language (e.g., C) within another programming language (e.g., C#)._

3. **to marshall:** _To arrange or make it ready for use._

## REFERENCES

---

1. [Native interoperability best practices (Microsoft)](https://docs.microsoft.com/en-us/dotnet/standard/native-interop/best-practices)
2. [Foreign function interface (Wikipedia)](https://en.wikipedia.org/wiki/Foreign_function_interface)
3. [Handle (Wikipedia)](https://en.wikipedia.org/wiki/Handle_%28computing%29)
4. Linux Man Pages
5. [Clang Compiler User Manual](https://clang.llvm.org/docs/UsersManual.html)
6. [Using the GNU Compiler Collection (GCC)](https://gcc.gnu.org/onlinedocs/gcc/)
7. [Position Independent Code on i386 -- what is it, why is it needed, how to write, how to exploit](https://web.archive.org/web/20160315011254/http://www.greyhat.ch/lab/downloads/pic.html)

---

Created 2022 by [Daniel](mailto:d849266@gmail.com)
