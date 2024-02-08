---
title: "Nice Programming Languages"
date: 2023-05-12T16:16:07+02:00
draft: true
---
# Some programming languages


`Nim`, `Zig` and `Crystal` are languages based on the `LLVM` compiler - similar to `GO` and `rust`. 

`Nim` should somewhat be similar to `Python` but can compile to a static program. 

`Zig` should somewhat be a `C` replacement, its compiler can in fact compile C to multiple architectures. 

`Crystal` should somewhat be similar to `Ruby` (similar to `Nim` and `Python`). 

`Elixir` is a functional programming language to the `Erlang` VM `Beam`. 

`Haskell` is another functional programming language. 


## *Nim*
   https://nim-lang.org/

{{< highlight nim >}}
echo "Hello World!"
{{< / highlight >}}


## *Zig*
   https://ziglang.org/
{{< highlight zig >}}
const std = @import("std");

pub fn main() void {
    std.debug.print("Hello, {s}!\n", .{"World"});
}
{{< / highlight >}}

## *Crystal*
   https://crystal-lang.org/

{{< highlight crystal >}}
puts "Hello World!"
{{< / highlight >}}

## *Elixir*
   https://elixir-lang.org/

{{< highlight elixir >}}
IO.puts("Hello, World!") 
{{< / highlight >}}

## *Haskell*
   https://www.haskell.org/

{{< highlight haskell >}}
main :: IO ()
main = putStrLn "Hello, World!"
{{< / highlight >}}



