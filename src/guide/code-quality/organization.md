# Organization

Organization is critical to code structure. Different parts of your code need to go in different files so they can make sense. This is actually a big reason why Java is such a great language to learn; Java almost forces you to organize things into different directories. However, in Vex we mostly use C++, so lets look at a few of the tools C++ gives us to work with.

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

It is also common to see a mix-and-match approach, particularly with `.cc` and `.hpp`, but I prefer to avoid this, as it just looks nicer to keep the extensions similar, but really that is up to personal preference.