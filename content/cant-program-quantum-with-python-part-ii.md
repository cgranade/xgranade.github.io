+++
title = "You Can't Program Quantum Computers With Python (Part II)"
date = "2023-07-07"

[taxonomies]
tags = ["quantum"]
+++

Earlier, I shared [Part I of my spicy take that "you can't program quantum computers with Python,"](https://buttondown.email/xgranade/archive/you-cant-program-quantum-computers-with-python/) focusing on ways to make the popular dichotomy between interpreted and compiled languages more precise and relevant. In particular, I made the claim that what was more relevant than being "compiled" was providing minimal runtime dependencies and providing strong design-time safety.

At least at the outset, that sounds like it's quite contradictory to how Python works as a language, and yet Python seems to be used quite well to program quantum devices --- what gives? In this part, I'll explore that tension by looking at two more issues. First, there's a difference between using Python as a programming language and using Python as a _metaprogramming_ framework. Second, people mostly aren't really writing quantum programs --- not yet.

# Programming and Metaprogramming

Consider a Python snippet like the following:

```python
def f(x):
    return x + 2
```

This would, at the outset, seem to be perfectly clear: it defines a function `f` that takes an argument `x` and adds it to the constant value `2`. In that sense, we're using Python as a programming language; that is, as a language to define a computer program for running on a classical computer.

We can do something different, though, with the same kind of function definition, but passing in something other than a number. Let's make a couple new data classes and see what we can do with them:

```python
from __future__ import annotations
from dataclasses import dataclass

@dataclass
class Var:
    name: str

    def __add__(self, other: Expr) -> Expr:
        return PlusExpr(self, other)

    # Needed so we can override what happens when we add Var
    # to a number, something like an int or a float.
    def __radd__(self, other: Expr) -> Expr:
        return PlusExpr(other, self)

@dataclass
class PlusExpr:
    left: Expr
    right: Expr

    def __add__(self, other: Expr) -> Expr:
        return PlusExpr(self, other)

    def __radd__(self, other: Expr) -> Expr:
        return PlusExpr(other, self)

NumberLiteral = int | float
Expr = Var | NumberLiteral | PlusExpr
```

(For the rest of the post, I'll quote snippets of this example rather than showing the full thing. If you want to see the entire example, check out [this gist on GitHub](https://gist.github.com/cgranade/136b806ba24b9efdc3fec70887ae8645).)

Now, if we call `f` with an instance of `Var`, what we get is a kind of description of a program:

```python
>>> x = Var("x")
>>> f(x)
PlusExpr(left=Var(name='x'), right=2)
```

Critically, when we call `f`, our program doesn't actually add anything at all, but only generates a description of a program. We could later use that description to actually run our program:

```python
>>> y = 2 * x + 1
>>> evaluate(y, x=3)
7
>>> evaluate(y, x=4)
9
```

Effectively, what we've done is to build up a Python object representing a program rather than build a program directly. If this sounds familiar, that's the same technique used by TensorFlow and other machine learning libraries for Python to compile expressions to various accelerator backends. Having the complete structure of an algebraic expression as a Python object makes it much easier to target different backends, but it also makes it much easier to manipulate expressions and perform different "passes." You can even do things like take the derivative of an expression:

```python
>>> x = Var("x")
>>> y = Var("y")
>>> z = 2 * x * x + 3 * x * y + (-4) * y
>>> simplify(derivative(z, x))
PlusExpr(left=PlusExpr(left=TimesExpr(left=2, right=Var(name='x')), right=TimesExpr(left=2, right=Var(name='x'))), right=TimesExpr(left=3, right=Var(name='y')))
>>> evaluate(derivative(z, x), x=3, y=4)
24
>>> evaluate(4 * x + 3 * y, x=3, y=4)
24
```

This only works because `z` is a Python object that we can manipulate, call methods on, and that we can modify. In general, programming techniques that work by manipulating and transforming other programs are known as _metaprogramming_ techniques; here, we've used Python's operator overloading as a basic kind of metaprogramming, but many other common techniques such as templates/generics, macros, and code generation broadly fall under the term metaprogramming.

By contrast, if `z` was declared as a Python function — if we programmed `z` directly — we'd have limited access to its internal structure and would only be able to _run_ `z` as a function (this is only mostly true, given that Python includes a disassembler, but that's a far more complicated approach to metaprogramming than what we're concerned with here).

One common application of metaprogramming is to quickly design new programming languages embedded in some host language. These new languages, sometimes called embedded domain-specific languages or embedded DSLs, borrow syntax from their host, but apply that syntax in distinct enough ways that programming in the embedded DSL can feel quite distinct from programming in the host language.

Suppose, for instance, that we want to generate some HTML programmatically using Python. We could then consider making an HTML-like language that embeds into Python — let's call it PML for Python Markup Language.

```python
>>> html = PmlNodeKind("html")
>>> body = PmlNodeKind("body")
>>> p = PmlNodeKind("p")
>>> a = PmlNodeKind("a")
>>> html(
...     body(
...         p("Hello, world!"),
...         p(
...             "Click ",
...             a("here", href="http://example.com"),
...             " to learn more."
...         )
...     )
... ).to_html()
'<html><body><p>Hello, world!</p><p>Click <a href="http://example.com">here</a> to learn more.</p></body></html>'
```

Here, function calls no longer mean what they usually mean in Python; rather, we've reused function calls to mean something much more _declarative_. In particular, `p("Hello!")` isn't read as "run a function `p` with `"Hello!"` as its argument," but as "add a new `p` tag with `"Hello!"` as its contents." Behind the scenes, the `PmlNode` class overloads `__call__` to implement that declarative meaning, but writing code in PML no longer really _feels_ like Python.

Many quantum libraries use this trick to build up circuits as well — let's look at a quick sample of building a quantum circuit using QuTiP:

```python
import qutip_qip as qp
import qutip_qip.circuit
circ = qp.circuit.QubitCircuit(2)
circ.add_gate("SNOT", 0)
circ.add_gate("CNOT", 1, 0)
```

The same way as we used our little PML example to generate HTML, QuTiP can generate OpenQASM 2.0 text from a circuit description:

```python
>>> print(qp.qasm.circuit_to_qasm_str(circ))
// QASM 2.0 file generated by QuTiP

OPENQASM 2.0;
include "qelib1.inc";

qreg q[2];

h q[0];
cx q[0],q[1];
```

Using this kind of approach, we can build up simple circuits like quantum teleportation. This time, let's use Qiskit to give it a try:

```python
qubits = QuantumRegister(3)
classical_bits = [ClassicalRegister(1) for _ in range(2)]
circ = qk.QuantumCircuit(qubits, *classical_bits)
prepare_entangled_state(circ, 1, 2)
unprepare_entangled_state(circ, 0, 1)
circ.measure(0, 0)
circ.measure(1, 1)
circ.z(2).c_if(classical_bits[0], 1)
circ.x(2).c_if(classical_bits[1], 1)
```

What's going on with that `.c_if` method? That gets to the heart of the difference between using Python to write quantum programs and using Python to implement an embedded DSL for writing quantum programs. The two approaches look very similar when we're writing quantum _circuits_, but couldn't be more different when we're writing quantum _programs_.

# Quantum Circuits versus Quantum Programs

Wait, but aren't quantum circuits the same as quantum programs? No, not really — _circuits_ consist of the special case of _nonadaptive_ quantum programs. That is, quantum programs where the list of quantum instructions to be executed is _fixed_, and does not depend on the outcomes of quantum measurements. Some circuit representations, such as that used by OpenQASM 2.0, include some small special cases of adaptivity, such as the teleportation example above, but for the most part, circuits are an almost vanishingly small subset of quantum programs in general. Due to hardware limitations with most prototype devices up to this point, though, circuits  have been where the vast majority of the effort in programming quantum devices has focused so far.

More generally, quantum circuits are interesting subroutines in larger quantum programs that include lots of control flow, including branching on the results of quantum measurement. To represent that control flow in an embedded DSL using metaprogramming, we have a challenge that we can't actually rely on the host language for control flow.

If we could, we might expect something like the following to work:

```python
# WARNING: this snippet is not valid!
qubits = QuantumRegister(3)
classical_bits = [ClassicalRegister(1) for _ in range(2)]
circ = qk.QuantumCircuit(qubits, *classical_bits)
prepare_entangled_state(circ, 1, 2)
unprepare_entangled_state(circ, 0, 1)
circ.measure(0, 0)
circ.measure(1, 1)
if classical_bits[0] == 1:
    circ.z(2)
if classical_bits[1] == 1:
    circ.x(2)
```

Indeed, that's closer to how standalone domain specific languages (that is, DSLs that aren't embedded into host languages) such as Q# or OpenQASM 3.0 represent conditional quantum operations. In Qiskit and other embedded DSLs, though, the `if` keyword is taken by the host language, not the embedded language. In the above attempt, what we actually get is a Python program that _either_ generates a quantum program with a `z` gate acting on qubit 2, or generates a quantum program without that gate. That is, the `if` statement is resolved _when we generate_ the quantum program, _not_ when we _run_ it.

Instead, Qiskit provides a `c_if` method that transforms part of a quantum program into a new quantum program that includes a classical condition, similar to how our earlier `derivative` function transformed one program into another. Other embedded DSLs, such as PyQuil, provide methods such as [`if_then`](https://pyquil-docs.rigetti.com/en/stable/apidocs/pyquil.quil.html#pyquil.quil.Program.if_then) and [`while_do`](https://pyquil-docs.rigetti.com/en/stable/apidocs/pyquil.quil.html#pyquil.quil.Program.while_do) to emit if-conditions or while-loops into quantum programs.

There are some ways around this challenge; for example, [QCOR](https://aide-qc.github.io/deploy/users/quantum_kernels/) uses Python's built-in disassembler to turn Python code into quantum programs:

```python
@qjit
def qpe(q : qreg):
    ...
    for i in range(bitPrecision):
        for j in range(1<<i):
            oracle.ctrl(q[i], q)
```

In general, though, implementing an embedded DSL for quantum programming within Python means that Python syntax is reserved for _metaprogramming_, and you need to come up with new ways of expressing _programming_ constructs like loops and conditions.

# Conclusions

With all that given, we now have enough to come back to the original spicy take and cool it down a bit. You can't use Python to write quantum programs, but you absolutely can use Python to write embedded programming languages that you can then use to write quantum programs. That does present some challenges compared to standalone languages like Q# and OpenQASM 3.0, but at the same time, it can be a good path for bringing the power and flexibility of Python into quantum programming as a metaprogramming engine.

