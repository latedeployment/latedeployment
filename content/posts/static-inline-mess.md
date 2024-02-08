---
title: "Static Inline Mess"
date: 2015-09-19T19:50:58+02:00
---

It's a known fact (at least to me), that `static inline` functions in C++ will generate multiple instances of a function if used in a different translation unit (a file, why call it differently?). 

But what about a `static` variable **inside** the `static inline` function? What's going on in the initialization of the variable?

If we use a function call inside the initialization of a static variable (`static int x = Foo();`), will `foo` be called once or the number of times the function is being used?

Let's see.

File foo.h:
{{< highlight cpp >}}
#include "grab.h"
void bar();
static inline int foo()
{
    static int ret = GrabTheNumber();
    return ret++;
}
{{< / highlight >}}
File grab.h:
{{< highlight cpp >}}
int GrabTheNumber();
{{< / highlight >}}


File main.cc:
{{< highlight cpp >}}
#include <stdio.h>
#include "foo.h"

int main(int argc, char **argv)
{
    foo(); // main.cc translation unit
    bar(); // bar.cc translation unit
    return 0;
}

int GrabTheNumber()
{
    printf("in GrabTheNumber!\n");
    return 0xAABBCCDD;
}
{{< / highlight >}}

File bar.cc:
{{< highlight cpp >}}
#include "foo.h"

void bar()
{
    foo(); // new foo() function here
}
{{< /highlight >}}
Compile:

{{< highlight bash >}}
    $ clang++ main.cc bar.cc -o test
{{< / highlight >}} 

Run:

    in GrabTheNumber!
    in GrabTheNumber!

Well, `GrabTheNumber` was called **twice**. 

**but** why do you care? I don't. 





