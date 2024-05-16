# Abstraction

Abstraction is one of the most important concepts in programming. It allows you to think more about the purpose of your code, and less about how it works. Good code makes ample use of abstraction, as it allows the code's purpose to be understood from reading it, and it helps you to minimize bugs.

There are 2 main types of abstraction: Data abstraction and Control abstraction. We will take a look at how to use each.

## Data Abstraction

Data abstraction is a form of abstraction that involves using data structures to reduce the complexity of a piece of code. Combining related data together in data structures allows you to manage it more easily, reduce line count, and reduce confusion. To understand this better, lets examine this snippet of code that calculates a dot product:

```cpp
double x1 = 1;
double y1 = 2;
double x2 = 2;
double y2 = 1;

double v1dotv2 = x1 * x2 + y1 * y2;
```

This code stores the 2 vectors in 2 variables each. In this case, the relation between the 2 components of each vector is obvious, however it still leaves something to be desired in cleanliness. Lets now see a version which uses a structure to help.

```cpp
struct vec {
  double x;
  double y;
};

vec v1 = {1, 2};
vec v2 = {2, 1};

double xdoty = v1.x * v2.x + v1.y * v2.y;
```

In this example, data is clearly associated with its data structure, and initializing the structure is more straightforward. Another good use of data abstraction is with units. Lets take a look at this code:

```cpp
double length = 1.5;
std::cout << length * 2.54 << "cm" << std::endl;
```

You might notice a problem: what is this length? 1.5 inches? Feet? Meters? Furlongs? Nobody knows, at least not without digging through more code to try to find the unit type. Even if you do know the unit type, multiplying this number by constants in your code can make things messy, and leave a lot of room for mistakes. Lets take a look at how ReveilLib (and OkapiLib) solve this problem, with a custom data type.

```cpp
using namespace rev;

QLength length = 1.5_in; // Now we can clearly see in the initializer that this is in inches

std::cout << length.convert(centimeter) << "cm" << std::endl;
```

As you can see, this fairly simple data type encodes the units that are used (internally using meters), and can automatically perform dimensional analysis using names instead of hard-to-remember and harder-to-read constants. Another feature of this usage of abstraction is this case, where an operator is incorrect:

```cpp
double len1 = 1.5;
double len2 = 2;

double len3 = 1.5 * 2; // This * should be a +

std::cout << len3 << std::endl;
```

This will end up printing `3`, when you would otherwise expect a `1.5`, and unless you have a keen eye, you might spend hours trying to figure out why. Lets see what happens when you try this with `QLength`:

```cpp
QLength len1 = 1.5;
QLength len2 = 2;

QLength len3 = 1.5 * 2;

std::cout << len3.convert(inch) << std::endl;
```

In this case, the compiler will give you an error, since multiplying 2 `QLength`s gives you a `QArea`, which is not the same type. This will immediately lead you to the erroneous `*`, saving you time and a real headache.

As you can see, structs and classes are a lot more than just a grouping of code; they can help you simplify your code and make it easier to understand. Let's now take a look at some control abstraction.

## Control abstraction

Take a look at the following code, and without scrolling down, figure out what it does.

```cpp
float number = 2;
const float threehalfs = 1.5F;

float x2 = number * 0.5F;
float y = number;
long i = *(long *)&y;
i = 0x5f3759df - (i >> 1);
y = *(float *) &i;
y = y * (threehalfs - (x2 * y * y));
```

No idea? Thats okay; this code has no comments nor any good explaination of what it does, and figuring out how exactly it works takes a lot of time and possibly a degree in computer science. Fortunately, it becomes a lot clearer when you wrap it in a function:

```cpp
float inverse_sqrt(float number) {
  const float threehalfs = 1.5F;

  float x2 = number * 0.5F;
  float y = number;
  long i = *(long *)&y;
  i = 0x5f3759df - (i >> 1);
  y = *(float *) &i;
  y = y * (threehalfs - (x2 * y * y));

  return y;
}

float number = 2;
float y = inverse_sqrt(number);
```

Now it seems fairly obvious: this is just an implementation of the inverse square root function:

\\[ y=\frac{1}{\sqrt{\text{number}}} \\]

This is why breaking code down into smaller manageable chunks is useful; if the inverse square root code appeared inline in a function, you would probably have no idea what it is doing and get completely lost on how the function works. If, however, it was stored in its own function like above, you'd see the name of the function call and immediately be able to understand what it does, even if you don't know exactly how it does it. This code was actually used in *Quake*, in its own function thankfully, and is known as the fast inverse square root algorithm.

Another way you might want to use abstraction is for something like a dot product. Lets review the example from above:

```cpp
struct vec {
  double x;
  double y;
};

vec v1 = {1, 2};
vec v2 = {2, 1};

double xdoty = v1.x * v2.x + v1.y * v2.y;
```

Simple enough if you are just taking 1 dot product, however anything involving a lot of geometry (particularly odometry algorithms and drive algorithms) might take dozens of dot products every iteration, and one incorrect symbol can screw all of that up. Lets apply what we've learned to make this code better. First, we can convert it to use QLength and QArea:

```cpp
using namespace rev;
struct vec {
  QLength x;
  QLength y;
};

vec v1 = {1_in, 2_in};
vec v2 = {2_in, 1_in};

QArea xdoty = v1.x * v2.x + v1.y * v2.y;
```

Then we can move the dot product into its own function. Since we are dealing with a custom struct, we can make this a member function.

```cpp
using namespace rev;
struct vec {
  QLength x;
  QLength y;

  QArea dot(vec& rhs) {
    return this->x * rhs.x + this->y * rhs.y;
  }
};

vec v1 = {1_in, 2_in};
vec v2 = {2_in, 1_in};

QArea xdoty = v1.dot(v2);
```

As you can see, at a glance it is easier to tell what the purpose of this code is! Lets also add an absolute value function:

```cpp
using namespace rev;
struct vec {
  QLength x;
  QLength y;

  QArea dot(vec& rhs) {
    return this->x * rhs.x + this->y * rhs.y;
  }

  QLength abs() {
    return sqrt(this->dot(*this));
  }
};

vec v1 = {1_in, 2_in};
vec v2 = {2_in, 1_in};

QArea xdoty = v1.dot(v2);

QLength absv1 = v1.abs();
```

You can see how it is easy to understand what each method does, but understanding it all at the same time might be a bit more difficult, and abstracting code lets you abstract other code much more freely, making the entire program easier to understand.

---

References:

- [Abstraction in C++ - GeeksforGeeks](https://www.geeksforgeeks.org/abstraction-in-cpp/)
- [Fast inverse square root - GeeksforGeeks](https://www.geeksforgeeks.org/fast-inverse-square-root/)