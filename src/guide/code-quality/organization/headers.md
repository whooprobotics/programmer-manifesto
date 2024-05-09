# Header and Source Files

C++ has 2 basic types of files:
- Source files
  - Typically end in `.cc`, `.cpp`, `.cxx`, or, rarely, `.C` (note the capitalization) or `.c++`
  - Contain code that performs actions, such as function definitions and global variable definitions
  - Usually found in the `src` directory, or any of its subdirectories
- Header files 
  - Typically end in `.h`, `.hh`, `.hpp`, `.hxx`, `.H`, `.h++`, or occasionally, nothing at all
  - Contain class and struct definitions, as well as `extern` variable declarations
  - Usually found in the `include` directory, but occasionally found alongside its corresponding source file

To avoid ambiguity, I highly recommend avoiding using `.C`, `.H`, and `.h`, as well as nothing at all. You should also avoid `.C`, `.H`, `.h++`, and `.c++` due to incompatibilities between operating systems. For consistency, you should probably pick one of the following:

- `.cc` and `.hh`
- `.cpp` and `.hpp`
- `.cxx` and `.hxx`

It is also common to see a mix-and-match approach, particularly with `.cc` and `.hpp`, but I prefer to avoid this, as it just looks nicer to keep the extensions similar, but really that is up to personal preference. For the purposes of this book, I will be using `.cc` and `.hh` throughout the rest, but do know that this all applies no matter which pair of extensions you choose.

## Proper usage of headers and source

Splitting code between headers and source files is one of the hardest things for a new C++ programmer to wrap their head around. Here are the general uses of the two types of files.

### Corresponding headers and source files

A common practice is to have 1 header per source file. Typically, there are 3 ways to approach this, which don't really have names, but for the sake of this document I am going to give them names.

- The Mirror Method
  - Every source file has a header with the same path, but in `include`
  - `src/quux/mirror.cc` \\( \rightarrow \\) `include/quux/mirror.hh`
- The Library Method
  - Similar to the mirror method, except its all kept in a subdirectory
  - `src/quux/library.cc` \\( \rightarrow \\) `include/[Library Name]/quux/library.hh`
  - Typically a prelude header is placed in the `include/[Library Name]/` directory
  - ReveilLib uses this, since it is a library
- The Neighbor Method
  - Corresponding headers are kept right next to their source file
  - `src/quux/neighbor.cc` \\( \rightarrow \\) `src/quux/neighbor.hh`

I will be using the Mirror method for my examples, but you can see how things apply.

### Including headers

Typically, you will see at the top of a source file (or even at the top of header files) something that looks like this:

```cpp
#include <iostream>
#include "rev.hh"
```

What `include` directives do is copy the contents of the specified header file to their location, but you don't need to know the exact mechanics to understand what it does. The real purpose of this is that it enables a file that includes a header to access anything that the specified header declares. There are 2 main types of `include` directives.

A system header directive is usually used to include something from the standard library, which is a core part of the C++ language. These files are not found in your `include` directory, but rather wherever your chosen compiler on your chosen operating system stores them. For example, if you include `iostream`, you get to use `cout` to print to the terminal.

```cpp
/* src/main.cc */

#include <iostream>

int main() {
  std::cout << "Hello world!" << std::endl;

  return 0;
}
```

prints

```
Hello World!
```

A file header, on the other hand, is usually used to include a header from your own project, or from a library you are using. For example,

```cpp
/* include/foo.hh */
/* Note: this file contains a terrible practice that is defining a function in a header. More on this later. */

#include <iostream>

void print_hello_world() {
  std::cout << "Hello world!" << std::endl;
}
```
alongside

```cpp
/* src/main.cc */

#include "foo.hh"

int main() {
  print_hello_world();
}
```

will print

```
Hello World!
```

Typically, a source file will include

- System headers
- Its own corresponding header
- File headers that are needed

### Include guards

As mentioned before, when a header is included, it copies its full contents. This can include more include directives which include more headers and so on. There is, however, a problem with this. If you have multiple headers which depend on the same header, and all of them are included, and they all include the header they depend on, you might end up with multiple copies of the same header file being imported. This is almost always a bad thing, but fortunately we have a solution: include guards.

There are 2 types of include guards. The first and most common type uses preprocessor directives to tell the compiler to omit the contents of the file if they've already been included:

```cpp
/* include/bar/baz.hh */
#ifndef INCLUDE_BAR_BAZ
#define INCLUDE_BAR_BAZ
/* File contents here */
#endif
```

This is common for a reason: it has worked for a long time and works with every ISO-compliant C++ compiler. However, it can be a little bit bulky, so theres an alternative.

```cpp
/* include/bar/baz.hh */
#pragma once
/* File contents here */
```

This solution can theoretically be faster on modern compilers, but officially it is not part of the ISO C++ spec. However, for the purposes of Vex, you will never encounter a compiler that doesn't support `#pragma once`, and even in the real world its pretty unlikely.

### Declarations 

A declaration is where you essentially tell the compiler "hey, this exists." For a function, this might look like this:

```cpp
int foo(int bar); /* Notice the lack of a code block */
```

This is not a definition; the function does not have an attached code block. As such, if you try to call it, the compiler will give you an error. We will see how to attach a declaration to this soon.

Global variables also have a way of being declared, without being defined.

```cpp
extern int baz;
```

The `extern` keyword here is... well, key. It tells the compiler "this exists already, its defined elsewhere, so you don't need to allocate memory for it." This is very important, because if you declare 2 global variables with the same name, the compiler won't know which is which, and even if that worked, you'd be in a tight spot, since your 2 files would be accessing different variables.

Generally, declarations belong in header files, while definitions go in the corresponding source file. Lets take a look at definitions now.

### Definitions and Initialization

A definition is where you actually specify the thing you declared earlier. Initialization is where you set the initial value of a global variable. 

Here is an example of a function definition:

```cpp
/* src/foo.cc */

int foo() {
  return 0;
}
```

As you can see, this function declaration has a code block associated with it, and so when `foo()` is called, the code block will run, and in this case, it will return `0`. Now for the cool part: you can combine the declaration and the definition together. Take, for example, the following program:

```cpp
/* src/foo.cc */

int foo() {
  return 3;
}
```

```cpp
/* src/main.cc */

int main() {
  std::cout << foo() << std::endl;
}
```

This fails to compile, because the compiler, while reading `main.cc`, can't find the function `foo`. However if we modify `main.cc` as follows:

```cpp
/* src/main.cc */
int foo();

int main() {
  std::cout << foo() << std::endl;
}
```

We get a terminal output:

```
3
```

That means that the declaration in `main.cc` "linked" to the definition in `foo.cc`. If we reorganize this to use header files, we can get something like this:

```cpp
/* include/foo.hh */
int foo();
```

```cpp
/* src/foo.cc */
#include "foo.hh" // Not strictly required, but this should be included for best practice

int foo() {
  return 3;
}
```

```cpp
/* src/main.cc */
#include "foo.hh"

int main() {
  std::cout << foo() << std::endl;
}
```

This compiles and gives us the same result!

As for global variable definitions, they are similar, except a valid global variable definition looks like this:

```cpp
int bar;
```

Thats all. You don't have to set it equal to anything; the compiler will just allocate it a bit of space (in this case, 4 bytes), and it will work. It can be initialized later using something like this 

```cpp
bar = 2158;
```

or you can define and initialize it all in one go:

```cpp
int bar = 2158;
```

Generally, this is how things should be done; everything that isn't a class with a built in default constructor (such as a smart pointer) should be initialized when it is defined to a safe value, or the value you want it to be initialized to. Failure to do so may cause unpredictable behavior.

This all, of course, belongs in a source file. Lets take a look at how to use it:

```cpp
/* include/bar.hh */
extern int bar;
```

```cpp
/* src/foo.cc */
#include "bar.hh" // Not strictly required, but this should be included for best practice

int bar = 3;
```

```cpp
/* src/main.cc */
#include "foo.hh"

int main() {
  std::cout << bar << std::endl; // This prints our initial value of 3

  bar = 7; // Reassignment is similar to initialization

  std::cout << bar << std::endl; // This prints our new value of 7
}
```