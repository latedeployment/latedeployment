---
title: "Crashing Compilers"
date: 2015-11-11T19:51:04+02:00
draft: true
tags: ['clang', 'gcc', 'python']
---

# The compilers

Let's see if we can crash some compilers. 

{{< highlight bash >}}
python3 -c "print('{' * 99999)" > test.c && clang test.c
{{< / highlight >}}

`Clang` **SEGFAULT** but `GCC` has made it :)

{{< highlight bash >}}
python3 -c "print('*' * 999999)" > test.c && clang/gcc test.c
{{< / highlight >}}

Both `Clang` and `GCC` will **SEFGAULT**

Many more operators are killing Clang - `GCC` Just fails here

{{<highlight bash >}}
python3 -c "print('{' * 999999999)" > test && nodejs ./test
{{< / highlight >}}

Kills `nodejs`. 

{{< highlight bash >}}
python3 -c "print('<--' * 999999999)" > test && php ./test
{{< / highlight >}}

and `PHP` is dead. 

`GO` and `Haskell` won't die but will kill your machine. 

`Ruby` for some good reason is able to survive. 

`Java` is nice because it fails for no main class :)

## What it means?

**Nothing for now** - just parsers gone wild.

