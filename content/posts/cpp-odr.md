+++
title = "ODR (One Definition Rule) in C++23"
date = "2023-08-12"
author = "Xiahua Liu"
tags = ["C++"]
+++

This post talks about the odr (one defintion rule) rule in the C++23 standard draft. This posit is based on the N4917 draft that is published on 2022-09-05.

All text from the standard will have a quote block around, and my explanations (using informal language) are below next to the quote block.

First we need to understand the boundary between a declaration and a definition, according to the traditional way of thinking C++, the declaration introduces symbols with types, and definition allocates memory for the symbols declared in the same translation unit.

## Preamble

> An entity is a value, object, reference, structured binding, function, enumerator, type, class member, bit-field, template, template specialization, namespace, or pack.

This is the definition of **entities** in C++. Basically everything in C++ that is not an expression, is a entity.

> A name is an identifier (5.10),operator-function-id (12.4), literal-operator-id (12.6), or conversion-function-id (11.4.8.3).

Definition of **names** in C++. Please be aware that it is different from **entities**, a name is more like the label on a entity not the entity itself.

> Every name is introduced by a declaration, which is a
> - *declaration*, *block-declaration*, or *member-declaration* (9.1, 11.4),

This is the typical types of declarations, for examples:

```cpp
/* Declaration */
int a; // This declare name a
[[noreturn]] void f (); // Declare f() with noreturn attribute

/* Block Declaration */
enum A : int; // Opaque enum declaration
class B; // Declare class B
namespace C; //Declare namespace C

/* Member Declaration */
class D {
    static int a = 10; // Declare D::a
    virtual int foo(); // Declare virtual method D::foo()
}
```
> * *init-declarator* (9.3),

>Each *init-declarator* or *member-declarator* in a declaration is analyzed separately as if it were in a declaration by itself.

Example:

```cpp
int a, b, c=0; // Equals to int a, int b, int c = 0
struct S { /* */ };
S S, T; // Declare two instances of struct S
```

> * *identifier* in a structured binding declaration (9.6),

Structured binding declaration is new in C++ 2017.

```cpp
int a[2] = {1, 2}; 

auto [x, y] = a;    // creates e[2], copies a into e,
                    // then x refers to e[0], y refers to e[1]
auto& [xr, yr] = a; // xr refers to a[0], yr refers to a[1]
```

For more information, check [cppreference - Structured Binding Declaration](https://en.cppreference.com/w/cpp/language/structured_binding).

> * *init-capture* (7.5.5.3),

This is used in the lambda expression.

```cpp
auto z = [a = 42] (int b) { return a+b; }
```

> * *condition* with a *declarator* (8.1),

Basically you can declare a thing in a *condition* statement

```cpp
if (int a=1; a)
// This equals to
int a=1;
if(a)
```

> * *member-declarator* (11.4),

A declarator is the name being declared. 

```cpp
class A {
    int *b; // *b is a member-declarator 
}
```

> * *using-declarator* (9.9),

Example:

```cpp
enum class button {up, down};
struct S{
    using button::up; // button::up is the using declarator
    button b=up;
}
```

> * *parameter-declaration* (9.3.4.6),

This is the declaration that happens inside the `()` when declaring a function.

```cpp
int foo(int a) {
    return a;
} // Declaring parameter a
```

> * *type-parameter* (13.2)

Example:

```cpp
template<typename T> // Declare T
struct foo {
    T a;
}
```

> * *elaborated-type-specifier* that introduces a name (9.2.9.4),

Elaborated type specifiers may be used to refer to a previously-declared class name (class, struct, or union) or to a previously-declared enum name even if the name was hidden by a non-type declaration. They may also be used to declare new class names.

If the name lookup does not find a previously declared type name,

* The the elaborated-type-specifier is a declaration that introduces the class-name if both of the following are true:
    * the elaborated-type-specifier is introduced by the *class-key*.
    * *class-name* is an identifier
* Otherwise the program is ill-formed (a compile error is produced).

Example

```cpp
template <typename T>
struct Node {
    struct Node* Next; // OK: lookup of Node finds the injected-class-name
    struct Data* Data; // OK: declares type Data at global scope
                       // and also declares the data member Data
    friend class ::List; // error: cannot introduce a qualified name
    enum Kind* kind; // error: cannot introduce an enum
};
 
Data* p; // OK: struct Data has been declaredclass A { /* */};
```

`struct Data* Data` introduces the type Data at global scope also the data member data.

> * class-specifier (11.1),

This is how you declare a class name.

```cpp
class A { /* Members */ };
```

> * `enum-specifier` or `enumerator-definition` (9.7.1),

This is how you declare a enum name.

```cpp
enum E : int { /* Enum members */ };
```

> *exception-declaration* (14.1), or

This is used in the exception handler.

```cpp
throw "Help!";
try {
    // ...
} catch(const char* p) { // Declare p
    // handle character string exceptions here
}
```

> implicit declaration of an injected-class-name (11.1).

The injected-class-name is the unqualified name of a class within the scope of said class.

In a class template, the injected-class-name can be used either as a template name that refers to the current template, or as a class name that refers to the current instantiation.

In a class scope, the name of the current class is treated as if it were a public member name; this is called injected-class-name. The point of declaration of the name is immediately following the opening brace of the class definition.

```cpp
int X;
 
struct X
{
    void f()
    {
        X* p;   // OK. X refers to the injected-class-name
        ::X* q; // Error: name lookup finds a variable name, which hides the struct name
    }
};
```

Yes, if you define `class A`, in the class definition, there will be a name `class A` that is implicitly declared inside the definition braces to mask the outside identifiers with the same name.

> [Note 3: The interpretation of a *for-range-declaration* produces one or more of the above (8.6.5). — end note]

The



## Declaration

> A declaration (Clause 9) may (re)introduce one or more names and/or entities into a translation unit. If so, the declaration specifies the interpretation and semantic properties of these names. A declaration of an entity or typedef-name X is a redeclaration of X if another declaration of X is reachable from it (10.7). A declaration may also have effects including:

In C++, you can declare an entity or typename `X` more than one times, but then need to be consistent with each other and all following declarations have no effects.

Declaration itself have a limited range of effects, including:

* A **static** assertion.
* Controlling template instantiation.
* Guiding template argument deduction for constructors.
* Use of attributes.
* Nothing (empty declaration).

Each entity declared by a declaration is also defined by that declaration unless:

* It declares a functions without specifying the function's body.

Example:

```cpp
int foo(void); /* Declare a function without body */
```

* It contains the `extern` specifier or a *linkage-specifition* and neither an *initializer* nor a *function-body*.

Example:

```cpp
extern int foo; /* Foo can be used to denote names outside this translation unit */
                /* Foo is declared but not defined */
extern const int bar = 1; /* Bar is declared and defined */
extern "C" {
    double sqrt(double); /* sqrt has C linkage */
                         /* No function body so it is not defined */
}
```

* It declares a non-inline static data member in a class definition.

Example:

```cpp
struct foo { /* Struct foo is declared */
    static const int i = 10; /* Not a definition of foo::i */
    const int i; /* This is declaration and definition of foo::i */
};
```

* It declares a static data member outside a class definition and the variable was defined within the class with the `constexpr` specifier. (Deprecated, do not rely on this rule)

Example:

```cpp
struct A {
    static constexpr int n = 5; /* Definition (declaration in C++ 2014) */
};

constexpr int A::n; /* Redundant declaration (definition in C++ 2014) */
```

* It is an *elaborated-type-specifier*.

Example:

```cpp
struct s { int a; };

void g() {
    struct s; // Hide global struct s with a block scope declaration
    s* p; // Refer to local struct s
    struct s { char * p; }; // define local struct s
    struct s; // Redeclaration, has no effect
}
```

* It is an *opaque-enum-declaration*.

Example:

```cpp
enum A : int; // Opaque enum declaration, can be redeclared later
enum A : int {red, blue, yellow}; // Redeclared with enum specifier
```

* It is a *template-parameter*.

Example:

```cpp
template<class T> // T is declared as a template parameter
int foo(T* a);
```

*  It is a *parameter-declaration* in a function declarator that is not the declarator of a *function-definition*.

Example:

```cpp
int foo (int) const & noexcept;
```

* It is a `typedef` declaration.
* It is an *alias-declaration* (`using`).

PS: Both `typedef` and `using` do not introduce new types. And `using` in *alias-declaration* has the same semantics as `typedef`. So I just put `using` examples here to represent both cases:

```cpp
template<class T>
struct Alloc {};
 
template<class T>
using Vec = vector<T, Alloc<T>>; // type-id is vector<T, Alloc<T>>
 
Vec<int> v; // Vec<int> is the same as vector<int, Alloc<int>>
```

When the result of specializing an alias template is a dependent *template-id*, subsequent substitutions apply to that *template-id*:

```cpp
template<typename...>
using void_t = void;
 
template<typename T>
void_t<typename T::foo> f();
 
f<int>(); // error, int does not have a nested type foo
```

* It is a *using-declaration*.

Note this is different from `using` in alias-declaration above, in this case, `using` is used to introduce names into the namespace.

```cpp
enum class button {up, down};
struct S {
    using button:up;
    button b=up;
}
```

* It is a *deduction-guide*.

Example:

```cpp
template<class T, class D = int>
struct S {
    T data;
};
template<class U>
S(U) -> S<typename U::type>;
struct A {
    using type = short;
    operator type();
};
S x{A()}; // x is of type S<short, int>
```

* It is a *`static_assert`-declaration*.

It does not introduce new names, and only should be used just for diagnotic purposes.

Example:

```cpp
static_assert(sizeof(int) == sizeof(void*), "wrong pointer size");
```

* It is an *attribute-declaration*.

Note attributes can be tested by `__has_cpp_attribute( attribute-token )`.

```cpp
[[gnu::always_inline]] [[gnu::hot]] [[gnu::const]] [[nodiscard]]
inline int f(); // declare f with four attributes
 
[[gnu::always_inline, gnu::const, gnu::hot, nodiscard]]
int f(); // same as above, but uses a single attr specifier that contains four attributes
```


* It is an *empty-declaration*.

```cpp
; /* Empty declaration has no effect at all */
```

* It is a *using-directive*.

A `using` directive does not introduce any names.

```cpp
namespace A {
    int i;
    namespace B {
        namespace C {
            int i;
        }
        using namespace A::B::C;
        void f1() {
            i = 5;
            // OK, C::i visible in B and hides A::i
        }
    }
    namespace D {
        using namespace B; 
        using namespace C; // B::C not in D, cannot hide A::i
        void f2() {
            i = 5;
            // ambiguous, B::C::i or A::i?
        }
    }
    void f3() {
        i = 5;
        // uses A::i
    }
}
void f4() {
    i = 5;
    // error: neither i is visible
}
```

* It is an *using-enum-declaration*.

A *using-enum-declaration* is equivalent to a *using-declaration* for each enumerator.

A *using-enum-declaration* in **class scope** makes the enumerators of the named enumeration available via
member lookup.


```cpp
enum class fruit { orange, apple };
struct S {
    using enum fruit;
    // OK, introduces orange and apple into S
};
void f() {
    S s;
    s.orange;
    // OK, names fruit::orange
    S::orange;
    // OK, names fruit::orange
}
```

* It is a *template-declaration* whose *template-head* is not followed by either a *concept-definition* or a declaration that defines a function, a class, a variable, or a static data member.

* It is an **explicit** instantiation declaration, or It is an **explicit** specialization whose *declaration* is not a definition.

In some circumstances, C++ implementations **implicitly** define the default constructor, copy constructor, move constructor, copy assignment operator, move assignment operator, or destructor member functions.

Example, Given:

```cpp
#include <string>
    struct C {
        std::string s;
        // std::string is the standard library class (23.4)
    };
int main() {
    C a;
    C b = a;
    b = a;
}
```

the implementation will implicitly define functions to make the definition of C equivalent to:

```cpp
struct C {
    std::string s;
    C() : s() { }
    C(const C& x): s(x.s) { }
    C(C&& x): s(static_cast<std::string&&>(x.s)) { }
    //
    : s(std::move(x.s)) { }
    C& operator=(const C& x) { s = x.s; return *this; }
    C& operator=(C&& x) { s = static_cast<std::string&&>(x.s); return *this; }
    //
    { s = std::move(x.s); return *this; }
    ~C() { }
};
```

## One Definition Rule

Each of the following is termed a definable item:

* A class type
* An enumeration type.
* A function.
* A variable.
* A templated entity.
* A default argument for a parameter (for a function in a given scope), or
* A default template argument.

No translation unit shall contain more than one definition of any definable item.

An expression or conversion is potentially evaluated unless it is an unevaluated operand (7.2.3), a subexpression
thereof, or a conversion in an initialization or conversion sequence in such a context. The set of potential
results of an expression E is defined as follows: