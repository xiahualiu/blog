+++
title = "Forward Declaration in C++"
date = "2023-08-02"
author = "Xiahua Liu"
tags = ["C++"]
+++

Today I will talk about forward declaration and IWYU(include what you use).

Forward declaration is a special feature for C and C++. Before we talk about forward declaration we must know about why declaration exist in C/C++.

In C or C++, the declaration and definition are seperated. Think compiling as writing a book, declaring is like writing down the table of content and the defining is like populating the real content of the book. Between them is the linking process which links a symbol's declaration to its definition.

When the compiler sees a new declaration, it adds it to the known symbol list (also keeps some basic information about it), and later on when the compiler sees it again it will look up the table and find that name.

Declaration is important because if you don't declare it first the compiler will complain if it fails to look up for a name. For example the following code would not get compiled:

```cpp
template<typename T>
int foo(T a){
    /* compiler error: ::unimplemented' has not been declared */
    return ::unimplemented;
}
template<>
int foo<int>(int a){
    return 0;
}

int main(){
    return foo(10);
}
```

Even though `::unimplemented` is not really included in the final code (we called the specialized version in `main`), the compiler still reads it and raise an error.

However by adding the declaration to `unimplemented` we make this error go away:

```cpp
/* Declare unimplemented but provide no definition to it */
int unimplemented();

template<typename T>
int foo(T a){
    /* no error */
    return ::unimplemented;
}
template<>
int foo<int>(int a){
    return 0;
}

int main(){
    return foo(10);
}
```

Now it comes to the forward declaration and why we want to use it.

The simple answer is forward declaration reduces the compilation time. 

For example, assume we have a huge header file `foo.h` and its source file `foo.cpp`:

Pretend we have tons declarations and definitions in those two files.

In `foo.h`
```cpp
int foo(int a);
/* 10000 more function declarations */
```

In `foo.cpp`
```cpp
int foo(int a){
    return a+1;
}
/* 10000 more function definitions */
```

In our `main.cpp`, we can definitely use these symbol declarations provided by `foo.h` by `#include "foo.h"`.

But then `main.cpp` will be really really long because `#include "foo.h"` will be replaced by all content from `foo.h` by pre-processor. `main.cpp` in the end will be a long file for the compiler to process.

But it doesn't make any sense, because `foo.h` and `foo.cpp` will get compiled along side with `main.cpp`, why should the compiler read the same 10,000 lines again?

If we only need to use several functions from `foo.h`. We can just forward their declarations in `main.cpp` and rely on the linking process to link them to their definitions in `foo.cpp`.

```cpp
// No need for #include "foo.h"
int foo(int);

int main(){
    return foo(10);
}
```

In the cmake we need to do:

```cmake
add_executable(main main.cpp foo.cpp)
```

Forward declaration is very useful in almost all senarios in C++ and should be preferred over `#include` directive at any time.

Even though `#include` is a traditional way to declare a lot of symbols from another file at once but in C++, header files can be really hard to process because of template.

## Break Circular Include

Because forward declaration doesn't need you to actually include, you can use it to break circular include in rare cases.

However, when you see circular includes, it is usually a sign of bad software architecture design, so I suggest re-organize your software instead of using forward declaration as a workaround.