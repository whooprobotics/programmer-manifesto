# Namespaces

Namespaces are an extremely useful feature in C++. Like how folders allow you to sort files, namespaces allow you to sort functions, variables, and other declarations. The basic structure of a namespace is this:

```cpp
namespace foo {
  int bar;
  void baz();
}
```

Definitions can also be put inside a namespace, however namespace declarations like above are best kept in header files.

From outside of a namespace, there are 2 ways to access its internals. You can use the scope operator `::`, as follows:

```cpp
// Header includes omitted for brevity
void main() {
  std::cout << foo::bar << std::endl;
}
```

As discussed in the [Headers](./headers.md) section, this specific case will print a garbage number, since `bar` has not been initialized, but the point is that `bar` can be accessed using the scope operator.

The other way to access bar is with `using`.

```cpp
// Header includes omitted for brevity
using foo::bar;
void main() {
  std::cout << bar << std::endl;
}
```

This does the same thing, but with 1 caveat: it does not work if there is already a global variable with the name `bar`.

You can also `using` an entire namespace at a time:

```cpp
using namespace foo;
void main() {
  std::cout << bar << std::endl;
}
```

Similar syntax can be used to call functions that are inside a namespace, or define them. Do note, though, that you need to have the member function declared inside the namespace before you can do this!

```cpp
void foo::baz() {
  std::cout << "Hello world!" << std::endl;
}

void main() {
  foo::baz();
}
```

Namespaces can also be nested inside one another

```cpp
namespace foo {
  namespace bar {
    int baz = 69;
  }

  void quux() {
    // Since foo::quux is a member of foo, it can directly access members of foo
    std::cout << bar::baz << std::endl;
  }
}

// This global function has the same name, but is not in the namespace, so the compiler pretends it has its own name

void quux() {
  // And because it isn't in the namespace, it has to use the scope operator
  std::cout << foo::bar::baz << std::endl;
}
```

Generally, when you reference a variable or function name, C++ will check from the innermost scope outward, until it reaches the global scope. The insides of functions (and, in fact, any if statements, loops, or other code blocks inside the function) count too, so in the example of `foo::quux`, if you attemped to access a variable named `thud`, the compiler will search for:

1. A variable named `thud` inside the `quux` function
2. A variable named `thud` in the `foo` namespace
3. A variable named `thud` in the global namespace

However, lets say you have something like this:

```cpp
int thud = 2;
namespace foo {
  int thud = 1;
  
  void quux() {
    int thud = 0;
    std::cout << thud << std::endl; // This will print 0
  }
}
```

In this case, calling `quux` would cause the program to print `0`, since the first variable named `thud` that the compiler finds would be the local variable declared inside the function. If you want to access the global named `thud`, you can use the scope operator with nothing before it:

```cpp
int thud = 2;
namespace foo {
  int thud = 1;
  
  void quux() {
    int thud = 0;
    std::cout << ::thud << std::endl; // This will print 2
  }
}
```

More generally, a `::` with nothing before it tells the compiler that the variable or function we are looking for is relative to the global namespace. Therefore, to access `foo::thud`, we can do this:

```cpp
int thud = 2;
namespace foo {
  int thud = 1;
  
  void quux() {
    int thud = 0;
    std::cout << ::foo::thud << std::endl; // This will print 1
  }
}
```