+++
title = "C++ Coding Style - Include Order"
date = "2023-08-03"
author = "Xiahua Liu"
tags = ["C++", "C++ Coding Style"]
+++

This post illstrate the importance of the `#include` order and is based on [Google C++ Style Guide](https://google.github.io/styleguide/cppguide.html).

Generally speaking, whenever you write a `*.cpp` file (such as `foo.cpp`), your include section should looks like this:

```cpp
/* Copyright section copied from template */

/* Same name header files */
#include "foo.h"

/* C System headers */
#include <unistd.h>
#include <stdlib.h>

/* C++ STL headers */
#include <string>
#include <vector>

/* Project headers */
#include "base/basictypes.h"
```

You may think "wait, does it really matter?" in your head, but it is actually quite important for your project to adhere to this order in every source file.

**Why should you put the same name header first?** Because this will ensure your header file, or interface file, gets compiled on its own. And as a result the header is guaranteed to be **self contained**.

Consider the following example (a bad example):

!()[images/cpp_include_order.png]

Because `foo.h` didn't include all its dependancies, the users of `foo.h` are forced to include them in their code.

**Make your code self contained!**