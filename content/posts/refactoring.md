---
title: "Long Time Ago Refactoring Ideas"
date: 2024-02-07T16:13:26+02:00
tags: ['C', 'CMake', 'windows', 'linux', 'macos', 'kernel']
---

*Refactoring ideas suggested long time ago from previous jobs*

# Libraries are too tightly coupled with each other

Libraries should not depend on each other too much, otherwise you have to update all libraries which misses the entire idea of decoupling.  

# A *CI* must check libraries on all relevant system *ifdefs*   

On the basic infrastructure, there shouldn't be a fix unless all systems are present (windows, macOS, linux), or at least correctly ignored.

# Libraries must be separated and with their own version

Libraries should also be independent.
Code should be linked against a given library version and work. 
Linux can be compiled against ver 1.1, and Windows against 1.3 and still run.
    

# Make sure features are added with a relevant feature flag

*GNU* source code has resolved this 30 years ago. 
While this is kind of an ugly solution, it makes life much easier. 

We create a new file called *OurFeatures.h*

Each feature will be assigned with a new #define * __WITH_FEATURE_XXX__ 1*

Code will be written as follows: 

    void foo(...)
    {
        ...

        #if __WITH_FEATURE_XXX__
            // do feature XXX
        #else
            // nothing to be done here
        #endif
    }


This allows merging to different systems while the code can still diverge.  (Say for instance thce *Windows* system doesn't support some specific system call)
Features can be disabled, but the code can still be released. 

Note that *CMake* provides a simple solution for this: *[configure_file](https://cmake.org/cmake/help/latest/command/configure_file.html)*

# Operating system agnostic layer for drivers

As kernel drivers must be written for multiple different systems and they do not share the same low level API, there must be a generic layer the driver developers provide for multiple systems. 

While this is complex, it gives us the benefits of: 

1. Write the code once, only the API layer will change between operation systems. 
2. Kernel engineers can work on multiple systems as the code they usually work against is coming from the agnostic layer rather than directly. 
3. Specific operating system person can only implement the layers code and don't have to handle the rest, as it will just work

# Build should always be based on tags
You must build from 1.2.3 *tag* not 1.2.3 *branch*. 

# Versions
Take a look at [Semantic Versioning](https://semver.org/)

# Reproduciable builds
Take a look at [Reproducible builds](https://reproducible-builds.org/)

# Automation of branches creation

Make sure that if branches build requires some modification on Jenkins or something similar, make sure you automate it as well.

# Garbage collection of branches

Don't leave too many branches on *GIT* not merged. 
    
# Questions about *GIT* you should decide on
1. How do you search history correctly
2. What do you write inside a commit message
3. How merge must look like - either fast forward merge or regular merge

# *GIT* end of line issue

Make sure you have a solution for EOL on multiple systems. 

# Coding convention 

Linux kernel a script call *checkpatch.pl* which check commits whether they apply to the linux coding convention style, or not. 

Invent something similar. 

# Automation and benchmarking

Make sure to benchmark relevant code changes, don't wait for customer to report back the speed of your app has decreased. 
Make it part of the QA/Automation cycle, even better, add it to the *CI* somehow. 

