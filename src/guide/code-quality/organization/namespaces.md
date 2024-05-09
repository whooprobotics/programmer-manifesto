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