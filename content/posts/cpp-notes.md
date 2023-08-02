+++
title = "Some C++ Notes"
date = "2023-07-16"
author = "Xiahua Liu"
tags = ["C++"]
+++

Some random C++ notes I learned.

## `GCC` Search Path

Many people mistakingly think `gcc` searches `$PATH` for all `#include` preprocessors but it is actually not.

`gcc` has 3 ways to look for the included files. 

* Searching inside the standard system directories, which is hardwared into the `gcc` itself for a specific system. You can force `gcc` to only search in this way with the angle-bracket form `#include<file>`.

You can use

```bash
cpp -v /dev/null -o /dev/null
```

or 

```bash
gcc -v
```

to learn about the standard system directories on your system.

* Additional directories, this is done mostly by adding compile option `-Idir` to `gcc`, which cause `dir` to be searched **after** the current directory and **ahead** of the standard system directories.

* The same directory that the source file is in.

For more information on this topic, visit GNU document about it https://gcc.gnu.org/onlinedocs/cpp/Search-Path.html.

## Float Number Comparison

Float number comparison gets overlooked a lot in the real world, a lot people will use:

```cpp
float a, b;
if (a == b) {;/* Do something */}
if (a != b) {;/* Do something */}
if (a >= b) {;/* Do something */}
if (a <= b) {;/* Do something */}
```

in their code. But if you are familiar with float representation for values you will notice float does not represent a precise value. https://en.wikipedia.org/wiki/IEEE_754 

And the precision depends largely on the range where the float is in.

So in the real world practice, instead of using the above test conditions, use:

```cpp
#include<cmath>

float a, b;
if (std::fabs(a-b) < TOLERANCE) {;} /* TOLERANCE could be a float rvalue */
if (std::fabs(a-b) > TOLERANCE) {;} /* TOLERANCE could be a float rvalue */
if (a-b > TOLERANCE) {;} /* TOLERANCE could be a float rvalue */
if (a-b < TOLERANCE) {;} /* TOLERANCE could be a float rvalue */
```

By specifying an extra `TOLERANCE` value, the float comparison can now be done safely and the comparing results will be predicatable according to the float precision requirements.

A general version (using template):

```cpp
#include <iostream>
#include <cmath>

#define GLOBAL_EPSILON 0.001f

template<typename FloatType>
bool float_equal(const FloatType &f1, const FloatType &f2, const FloatType &eps);
template<typename FloatType>
bool float_equal(const FloatType &f1, const FloatType &f2);
template<>
bool float_equal<float>(const float &f1, const float &f2, const float &eps){
    return std::fabs(f1-f2)<eps;
}
template<>
bool float_equal<float>(const float &f1, const float &f2){
    return std::fabs(f1-f2)<GLOBAL_EPSILON;
}

int main()
{
    std::cout << float_equal(123.0f, 123.0f) << std::endl;
    return 0;
}
```

## Add Suffix (or Prefix) to Literals

This is very important because if you don't the compiler only uses the default types (`int` for integer literals and `double` for float literals). This could cause unwanted results:

```cpp
unsigned char a = 1u;
unsigned char b = 3u;
if (a+1 > b); // Compiler Warning: Compare unsigned to signed type.
```

The compiler will warn you about comparing a unsigned value to a signed value because `a+1` will be `int` instead of `unsigned char` anymore.

```cpp
unsigned char a = 1u;
unsigned char b = 3u;
if (a+1u > b); // No Compiler Warning, you did a great job!
```

Always put suffix to numeric literals, C++ is a strong type language, you should always explicitly declare types not only the variables but also those literals.

The list of available suffix and prefix for each type can be found below:

* Integer Literals. (https://en.cppreference.com/w/cpp/language/integer_literal)
* Float Literals. (https://en.cppreference.com/w/cpp/language/floating_literal)
* Char Literals. (https://en.cppreference.com/w/cpp/language/character_literal)
* String Literals. (https://en.cppreference.com/w/cpp/language/string_literal)