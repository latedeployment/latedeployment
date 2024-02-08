---
title: "C++ uint8_t std::cout"
date: 2015-11-20T19:51:17+02:00
draft: true
---
So, I just wanted to print a loop of numbers from 0 to 100 with `C++` and `std::cout`.

This is why `C++` template mechanism makes me giggle from time to time.

{{< highlight cpp >}}

for (uint8_t i = 0; i < 100; ++i)
{
    std::cout << i << std::endl;
}

{{< / highlight >}}
Well, the results (as expected by the `C++` spec writers) are the `ASCII` table from 0 to 100...

`uint8_t` is unsigned char which prints characters in std::cout...

Isn't it clever?



