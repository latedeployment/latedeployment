---
title: "Clang GCC Cuteness"
date: 2024-09-19T19:50:43+02:00
draft: true
tags: ['clang', 'gcc']
---

# Array initialization

{{< highlight cpp >}}
#include <stdio.h>
 
int arr[10] = {
        [0] = 5,
        [3] = 2,
        [4] = 5,
        [7] = 8
};
 
int main (int argc, char** argv)
{
    for (int i = 0; i < (sizeof (arr) / sizeof(arr[0])); ++i)
            printf ("[%d] == %d\n", i, arr[i]);
    return 0;
}

{{< / highlight >}}

# ',' Operator in if statements

{{< highlight cpp >}}
#include <stdio.h>
int main (int argc, char** argv)
{
    int a = 0;
    int b = 0; 
    int c = 0; 
    // will enter the if when a | b | c is nonzero because a+b+c evaluation
    if (printf ("A: %d ", a),printf ("B %d ", b),printf ("C %d ", c), a + b + c)
    {
           printf ("in first if\n");
    }
    // will enter the if because c=1 and c evaluation afterwards
    else if ((a = 1), (b = 1), (c = 1), c)
    {
        printf ("\nA: %d ", a), printf ("B %d ", b),printf ("C %d\n", c);
    }
 
    return 0;
}
{{< / highlight >}}

# C Destructor !
{{< highlight cpp >}}
#include <stdio.h>
#include <stdlib.h>

void int_release(int** dest);
#define int_dest_t __attribute__((cleanup(int_release)))

int main (int argc, char** argv)
{
    int_dest_t int *x = malloc(1024);
}

void int_release(int** dest)
{
    printf("freeing %p\n", *dest);
    free(*dest);
}

{{/ highlight >}}


