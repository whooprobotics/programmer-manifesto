# Classes

Classes are arguably the feature where C++ becomes C++. They are what makes C++ an object-oriented programming language. For you Java developers, this is probably the area you will be most comfortable with. Classes are very complex and so I can't cover everything about them, but I will try to get the basics of what they are and how to use them done here.

A class (or, almost equivalently, a struct) is a custom variable type, which can include multiple member variables, member functions (which we call methods), and even other classes/structs inside.

## Declaration

Like with namespaces, classes, alongside all of their members, should be declared in a header, however since they are part of an object and not in the global namespace, you don't need to include `extern`, and you don't need to define class members in the accompanying source file. Let's take a look at an example:

```cpp
/* my-class.hh */
class MyClass {
 public:
  static void foo();
 private:
  int bar {0}; // Initializing in the header file is okay here, since these are members of the class, and don't "really" exist
};
```

Definitions should be kept to the source file

```cpp
/* my-class.cc */
#include <iostream>
#include "my-class.hh"

void MyClass::foo() { // Use the scope operator to define the method
  std::cout << bar << std::endl;
}
```

## Static members and permissions

A static member of a class has 2 important properties:

- They are global, not real class members
  - Therefore, anything can access these functions and variables, *without needing an instance of the class*
- They cannot access non-static member variables of the class
  - This is because the static member, since it doesn't need an instance to be used, also can't know what instance it might get to use.

Static members of classes behave almost identically to the way they would if the class was a namespace instead, except they can also access private class members. Lets see an example:

```cpp
class MyClass {
 public:
  static void foo() {
    std::cout << bar << std::endl;
  }
 private:
  int bar = 0;
}; // This semicolon is important!
```

This works, and can be called from anywhere in the program, however other parts of the program *cannot* directly read `bar`. Similarly, any static functions declared in the `private` area can't be accessed outside of the class (except by `friend` functions, but these are an advanced topic that I will not be covering).

One other thing you should know is that when declaring a static variable in a class declaration (which should be in a header), you should not use the `extern` keyword.

## Class instances

An instance of a class is similar to an instance of any other variable. Similar to integers, if you have a class called `MyClass`, you can have multiple independent instances of the class, and these instances will not share member variables.

## Member methods

As discussed above, a class member method is basically just a function inside the class. Non-static member methods can access other non-static methods within the class, as well as instance variables, and, like other functions, they can access anything global, including everything in the class's static namespace.

To access instance variables or methods in a non-static method, you can just reference them by name. To access them outside of the method, you have to use the `.` operator.

```cpp
class MyClass {
 public:
  void foo();

  void bar() {
    foo(); // This is allowed because bar is non-static
  }
};

void MyClass::foo() { // Notice this is non-static
  // Code here
}

void MyClass::bar() { // Notice this is non-static
  foo(); // This can directly call foo
}

int main() {

  MyClass::foo(); // This fails because `foo` is not static, and so it needs an instance to work with

  MyClass instance; // Class name followed by variable name, just like declaring an int
  instance.foo(); // Call foo
  return 0;
}
```

If you want to be more specific (which is highly recommended), you can use the special `this` pointer with the `->` operator to access instance variables and methods inside a method.

```cpp
void MyClass::bar() {
  this->foo(); // This works too
}
```