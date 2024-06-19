# Styling

Another very important concept in programming is styling. Take a look at the following code.

```cpp
void moveTo_target(double X, double Y) {
  auto Angle= atan2(X,Y);
  double d = sqrt( pow(X,2)+pow(Y, 
    2));
  goTo(
    d,Angle)
}
```

Notice how difficult that code is to read. Now lets examine a cleaner version of the code that accomplishes essentially the same purpose (though with some naming differences)

```cpp
void move_to_target(double x, double y) {
  double angle = atan2(x, y);
  double distance = sqrt(x*x + y*y);
  go_to(distance, angle)
}
```

This code is easier to read for a few reasons. The whitespace is consistent, the way variables and functions are named is consistent, and things are named appropriately. By far the most important concepts in styling code can be summed up by 2 points

1. Name things well
2. Use consistent capitalization and whitespace throughout a project

Outside of these things, there really isn't one style guide that is the best. For the purposes of working with Aggie Robotics code, it is best to follow the established style guide, which is the same as the Chromium style guide, with a few exceptions in capitalization:

| Item                     | Convention             |
| ------------------------ | ---------------------- |
| Struct/Class names       | `UpperCamelCase`       |
| Functions & Methods      | `snake_case`           |
| Local & Member Variables | `snake_case`           |
| Constants                | `SCREAMING_SNAKE_CASE` |

If anything is unclear, it is best to refer to existing code within the project to find something similar.

> [!TIP]
> Use `clang-format` and `clang-tidy` to enforce styling


## Naming Things

There is an infamous joke in computer science:

> There are only two hard things in Computer Science: cache invalidation and naming things.
>
> -- Phil Karlton

Fortunately for us, right now you only need to learn how to name things. 

One of the biggest sins when naming things is the dreaded single-letter variable name. Its something we all do when we are learning to program; we name variables using a single letter, then have no idea what the code does when we come back to it. Take a look at the following code:

```cpp
void TRIO::set(Position p) {
  m.take(TIMEOUT_MAX);

  this->p.p = p;
  this->p.v = {0 * i / s, 0 * i / s,
                          0 * i / s};

  this->h_i = sensor.get() - this->p.theta.convert(d);

  m.give();
}
```

What does it do? Good question; its hard to take a glance at that code and understand exactly what it does. Heres the same code with meaningful names:

```cpp
void TwoRotationInertialOdometry::set_position(Position pos) {
  current_position_mutex.take(TIMEOUT_MAX);

  current_position.pos = pos;
  current_position.vel = {0 * inch / second, 0 * inch / second,
                          0 * radian / second};

  this->heading_ticks_init = inertial.get_heading() - current_position.pos.theta.convert(degree);

  current_position_mutex.give();
}
```

Now you can see pretty clearly what this code does. It sets the current position, and sets the zero point of the heading for the gyro. This isn't just an example; this code is actually used in ReveilLib's TwoRotationGyroOdometry.

There are however a few exceptions where it is okay to use single letter variable names. An example is in some `for` loops, like this one

```cpp
for(int i = 0; i < 10; i++) {
  std::cout << "foo" << std::endl;
}
```

This `for` loop is pretty simple; it just prints `foo` 10 times. Notably, `i` is meaningless, and it doesn't take on a meaning. Lets examine the following code:

```cpp
StationStatus check_all_stations() {
  for(int i = 0; i < 8; i++)
    if(get_station_status(i) == StationStatus::DOWN)
      return StationStatus::DOWN;

  return StationStatus::OK;
}
```

While this code isn't awful, at a glance `i` still does not have a super well-defined meaning. This can be fixed by naming it appropriately for what its used for:

```cpp
StationStatus check_all_stations() {
  for(int station_id = 0; station_id < 8; station_id++)
    if(get_station_status(station_id) == StationStatus::DOWN)
      return StationStatus::DOWN;

  return StationStatus::OK;
}
```

> [!Note]
> If you only have a single statement, you don't have to surround code in a for loop or if statement with `{}`, which is why the above code works. However, this should be used sparingly,
> since omitting these can make code harder to read sometimes.

This code makes it much clearer what is happening with the iterator variable. Generally, a good rule of thumb is to try to give every variable a clear meaning in its name, such that if someone whos never seen this code before looks at it, they can immediately understand what its for. It is okay to use meaningless variable names, but only when the variable itsself has no meaning.

## Whitespace

Whitespace is also an important thing to remember in your code. While cool one-liners and compact code look cool sometimes, nobody wants to maintain that code, so having whitespace is important. Here are a few key rules to keep in mind:

*Code nested inside a codeblock should be indented.* This means
- Anything inside a function
- Anything inside a for or while loop
- Anything in an if/else statement

Python is a great way to learn about indentation, since it requires correct indentation to run.

Another important concept is just spaces between things. Code needs to have spaces to be readable. For example

```cpp
distance = speed * time;
```

is just way more readable than

```cpp
distance=speed*time;
```

for the same reason that we put spaces between words in English. Youwouldn'twanttoreadawholebookwiththisspacing,wouldyou?

Finally, use of the newline needs to be considered. Generally, outside of the start of a for loop (you know, `for(int i = 0; i < 10; i++)`), almost every semicolon should be followed by a newline. Anything inside of a real codeblock also needs to be on a newline, and really for the most part, you've probably already been doing all of this, since even bad code tends to use newlines appropriately. When you're in doubt, just use `clang-format`.

