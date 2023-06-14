+++
title = "You Can't Program Quantum Computers With Python (Part I)"
date = 2023-06-13

[taxonomies]
tags = ["quantum"]
+++

I'll warrant that "you can't program quantum computers with Python" is a [spicy take on quantum computing](https://wandering.shop/@xgranade/110520986208758253), given the prevalence of Python-based toolchains for quantum computing — everything from [QuTiP](https://qutip.org/) through to [Qiskit](https://qiskit.org/) offer to allow Python users a way to write and run quantum programs. At the same time it's not _that_ hot a take, as the same qualities that prevent using Python to write quantum programs perhaps paradoxically also make Python a great language for quantum computing.

To reconcile those two seemingly opposite claims, we'll need to take a tour through two long-running dichotomies in classical software development: interpreted versus compiled languages, and programming versus metaprogramming. That will necessarily be a bit long for a single post, so let's dive in with a discussion of compiled versus interpreted languages.

# Compiled and Interpreted Languages

Generally, when we talk about programming languages, folks tend to separate them into compiled languages like C, C++, Rust, and Go, or interpreted languages like Python and JavaScript. If you ask about how to classify languages like C# or Java that use a virtual machine to interpret intermediate-level bytecode at runtime, you'll get different answer depending on the biases and preferences of whomever you ask. Add just-in-time compilation into the mix, and you'll as often or not get demure mumbles followed by a sudden shift to talking about the weather.

Taxonomy is hard, and fitting all languages into one of two buckets is one of the hardest taxonomical debates we run into in classical software development. So let's approach the problem with overwhelming and embarrassing levels of hubris, and simply solve it: **all programming languages are interpreted, whether or not they involve the use of a compiler**. The interpreter may be built into your CPU, or might be a complex userspace application, but it exists nonetheless. A sequence of bytes is always meaningless on its own, without reference to a particular device or application interpreting them as instructions.

Rather, I'd posit that when people refer to the division between compiled and interpreted languages, a large part of the confusion stems from that there's actually two distinct technical evaluations (at least!) being juggled behind the scenes: how complex are the runtime requirements for a language, and what design-time safety does a language provide? Both of these questions are quite tied up in the task of programming quantum computers, to put it mildly.

# Runtime Dependencies

When you write in a language like C, you have access to the _C Standard Library
_, a set of functions and data types like `fopen` for opening files, or `strncpy` for copying *n* bytes from one part of memory to another. Except when you don't. That standard library needs to exist at runtime, leading to a variety of different implementations being made available, including glibc, muslc, Android's libbionic, and Microsoft's Universal C Runtime. In some cases, such as working with embedded microcontrollers or other low-power devices, those kinds of implementations might not make sense, such that you might not have a standard library available at all. As a result, the programs you write in C  may have more or less runtime requirements, depending on what capabilities you assume and what kinds of devices you're trying to work with.

Traditionally, languages that we think of as being interpreted tend to have much heavier runtime requirements than compiled languages. JavaScript programs require either a browser or an engine like [NodeJS](https://nodejs.org/en) to run (except when [they don't](https://github.com/Moddable-OpenSource/moddable/blob/public/documentation/xs/XS%20Differences.md)), while Python carries the entire Python interpreter and its rather large standard library as dependencies.

Except when it doesn't. The [MicroPython project](https://micropython.org/) provides an extremely lightweight implementation of the Python interpreter and an optional compiler, allowing Python to be used on small, low-power microcontrollers. The tradeoff is that, just as programming in C without the standard library is harder than programming with it, the version of Python recognized by MicroPython is a strict subset of the Python most people are used to working with, leading to a [number of important differences](https://docs.micropython.org/en/latest/genrst/index.html).

Even MicroPython, though, is likely more than what can be reasonably run on the classical parts of a quantum device, especially when considering the extremely strict latency requirements imposed by coherence times. While there's no fixed set of runtime dependencies associated with a language, the fact that Python is very _dynamic_ makes it difficult to fit its dependencies within the exacting requirements of quantum execution.

What do we mean by dynamic, though? Luckily, there's more to this post!

# Design-Time Safety

In my [previous post on types and typeclasses](https://buttondown.email/xgranade/archive/subclasses-and-typeclasses/), I highlighted the role that types can play in checking the correctness of programs. To use an example from that post, consider passing an invalid input to a Python function that squares its argument:

```python
>>> def square(x):
...     return x * x
...
>>> print(square("circle"))
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "<stdin>", line 2, in square
TypeError: can't multiply sequence by non-int of type 'str'
```

Generally, if you make a logic error of this form in Python, it gets caught when you run the program. On the other hand, if I try to write the same thing in Rust, I get an error when I compile the program, complete with suggestions as to how to fix it:

```shell
$ cat src/main.rs
fn square<T: Num + Copy>(x: T) -> T {
    x * x
}

fn main() {
    println!("{}", square("hello"));
}
$ cargo build
error[E0277]: the trait bound `&str: Num` is not satisfied
 --> src/main.rs:8:27
  |
8 |     println!("{}", square("hello"));
  |                    ------ ^^^^^^^ the trait `Num` is not implemented for `&str`
  |                    |
  |                    required by a bound introduced by this call
  |
  = help: the following other types implement trait `Num`:
            BigInt
            BigUint
            Complex<T>
            Ratio<T>
            Wrapping<T>
            f32
            f64
            i128
          and 11 others
note: required by a bound in `square`
 --> src/main.rs:3:14
  |
3 | fn square<T: Num + Copy>(x: T) -> T {
  |              ^^^ required by this bound in `square`

For more information about this error, try `rustc --explain E0277`.
```

That isn't to say that Rust is _better_, so much as that it provides different tradeoffs. While Python is far more flexible, and requires developers to specify a lot less information up front, that also means that there's less information available to validate programs as we write them instead of when we run them. The dichotomy isn't as strict as that, of course, thanks to design-time validators for Python like Pylint and Mypy, but it is generally true that languages like Python provide less design-time guarantees while languages like Rust provide more.

Much of that difference stems from the fact that in Python, types are _dynamic_, meaning that they are a _runtime_ property of values. We can even check what type something is at runtime and make decisions accordingly:

```python
>>> if random.randrange(2):
...     x = 42
... else:
...     x = "the answer"
...
>>> type(x)
<class 'str'>
>>> print("str" if isinstance(x, str) else "not str")
str
```

Here again, the dichotomy between dynamic and static type systems is a bit hard to pin down, given that languages like C# and Java support _reflection_ as a way to make runtime decisions about types that would normally be static, and that polymorphism in C++ allows for some amount of dynamism subject to an inheritance bound. Even Rust has the `dyn` keyword for building a dynamic type out of a static typeclass bound. We can broadly say, though, that Python has a much more dynamic type system than most languages.

The practical effects of that dynamism are far-reaching, but what concerns us in this post is the impact on quantum computing: it's difficult to use types and other design-time programming concepts to make conclusions about what a block of Python code will do when we run it. That works well when computing time is cheaper than developer time, such that we can run and test code to ensure its validity, but works dramatically less well when computing time is expensive, as in the case of a quantum device. 

# Next time on...

In order to be useful for programming quantum computers, we want a language that is easy to use with minimal to no runtime dependencies, and that provides strong design-time guarantees as to correctness. Put together, exploring these two dichotomies tells us that we want something that looks more like what often gets called a compiled language, even if taking the compiled-vs-interpreted taxonomy literally isn't the most useful.

That then leaves the question as to what that "compiled" language should look like, if not Python. In the next post, I'll try to answer that by arguing that one very good alternative to using Python to program quantum computers is... to use Python.
