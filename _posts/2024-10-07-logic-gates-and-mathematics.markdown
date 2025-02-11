---
layout: post
title:  "Formalising Logic Gates As Mathematical Expressions"
date:   2024-10-07 00:19:13 +0100
categories: Mathematics
---
<style>
.tex sub, .latex sub, .latex sup {
  text-transform: uppercase;
}

.tex sub, .latex sub {
  vertical-align: -0.5ex;
  margin-left: -0.1667em;
  margin-right: -0.125em;
}

.tex, .latex, .tex sub, .latex sub {
  font-size: 1em;
}

.latex sup {
  font-size: 0.85em;
  vertical-align: 0.15em;
  margin-left: -0.36em;
  margin-right: -0.15em;
}

.formula {
    font-size: 30px; /* Adjust the size as needed */
    font-weight: bold;
}
</style>

# Premise
Many years ago when I was 16 I explored the idea of formalising logic gates (AND, OR, XOR etc.) as mathematical formulas. My idea was that by using a calculator with storage capabilities (in my case the spreadsheet functions of the [Casio FX991EX](https://www.amazon.co.uk/Casio-Fx-991ex-Scientific-Calculator-991/dp/B011UK5DGY)) and a program encoded as a mathematical formula it might be possible to program a non-programmable calculator to add new functions (such as solving new classes of maths problems).

<img src="https://m.media-amazon.com/images/I/61GgfWl3WLL._AC_UF1000,1000_QL80_.jpg" height=512>
<p><small> <i> The Calculator In Question </i> </small></p>

# Logic Gate Formulas
The core formulas I worked out were:
<center>
<span class=formula>AND(A,B) = A × B</span>
<br>
<span class=formula>OR(A,B)=A+B−(A×B)</span>
<br>
<span class=formula>NOT(A)=1−A</span>
<br>
<span class=formula>NAND(A,B)=1−(A×B)</span>
</center>

And from these formulas every other logic gate could be derived (since NAND can represent any other logic gate, though the others help optimise)

# Problems In Equivalence
There are a few equivalence problems that make these formulae imperfect in a simplistic sense. Firstly there is no way to feed a formula into itself effectively (which is needed for most forms of storage) and subsequently there is no way to produce complex behaviour such as loops without additional storage (though this wasn't the end of the world in the original context as I could store values in spreadsheet cells).

# Defining A Compiler From Logic Gates To Mathematical Expressions
In the past I defined a simple compiler that converted logic gates to these equivalent expressions, done using Common Lisp (which I was deeply into at the time). Having lost that compiler I decided to reinvent it in Python, which I chose due to SymPy, a symbolic expressions library, which I could use to optimise the resultant equations the compiler would produce, creating the smallest formulas/programs possible.

# Difficulties In Defining The Gates
The biggest issue is defining the gates/programs effectively as a single equation. Since there are multiple possible return values for some programs (such as half-adders which return a sum and a carry bit) we need two separate equations to find either path (unless we can solve some way of joining them smoothly, which I haven't solved here but would be a huge innovation).

The old compiler would take the gates as functions and simply evaluate them down. We will follow this strategy today, but it will make future implementation of delayed evaluation less pretty than in the Common Lisp solution.

A crude evaluator is below:
```python
def And(left, right):
    return f"{left} * {right}"

def Xor(left, right):
    return f"{left} + {right} - 2 * (A * B)"

half_adder1 = And("A", "B")
half_adder2 = Xor("A", "B")

print("Carry:", half_adder1)
print("Sum:", half_adder2)
```

Which results in:
```bash
Carry: A * B
Sum: A + B - 2 * (A * B)
```

We can confirm these formulas are correct, for example by adding 1 and 1 together (expecting 0 in the sum and 1 in the carry):

Carry = 1 * 1 = 1

Sum = 1 + 1 - 2 * (1 * 1)
Sum = 1 + 1 - 2 * 1
Sum = 1 + 1 - 2
Sum = 2 - 2
Sum = 0

So we can see that this basic code can produce a valid solution for a simple binary adder, representing it as a mathematical formula. Let's go further and produce optimised outputs using SymPy:

```python
from sympy.parsing.sympy_parser import parse_expr
from sympy import pretty

def SimplifyEq(eq):
    parsed_eq = parse_expr(eq)
    return pretty(parsed_eq, use_unicode=False)

def And(left, right):
    return f"{left} * {right}"

def Xor(left, right):
    return f"{left} + {right} - 2 * (A * B)"

half_adder1 = And("A", "B")
half_adder2 = Xor("A", "B")

print("Sum:", SimplifyEq(half_adder1))
print("Carry:", SimplifyEq(half_adder2))
```

Which produces:

```bash
Sum: A*B
Carry: -2*A*B + A + B
```

This seems minor but with a full-adder it can be a massive help.

# A Full Adder To Test Our Program
```python
from sympy.parsing.sympy_parser import parse_expr
from sympy import pretty

def SimplifyEq(eq):
    parsed_eq = parse_expr(eq)
    return parsed_eq

def And(left, right):
    return f"{left} * {right}"

def Xor(left, right):
    return f"{left} + {right} - 2 * (A * B)"

full_adder1 = Xor(Xor("A", "B"), "C")
full_adder2 = And(And("C", Xor("A", "B")), And("A", "B"))


print("Sum (raw):", full_adder1)
print("Carry (raw):", full_adder2)

print("Sum:", SimplifyEq(full_adder1))
print("Carry:", SimplifyEq(full_adder2))
```

This produces:

```bash
Sum (raw): A + B - 2 * (A * B) + C - 2 * (A * B)
Carry (raw): C * A + B - 2 * (A * B) * A * B

Sum: -4*A*B + A + B + C
Carry: -2*A**2*B**2 + A*C + B
```

From this we can see that we get a greatly smaller and simpler equation in the simplified condition, which helps save the time needed to write and input the resulting equations (the simplified forms are ugly, sadly SymPy refused to produce the nice unicode equations in my terminal).

Made pretty for carry it is:
<br>
<span class=formula><math xmlns="http://www.w3.org/1998/Math/MathML"><mo>-</mo><mn>2</mn><mo>*</mo><msup><mi>A</mi><mn>2</mn></msup><mo>*</mo><msup><mi>B</mi><mn>2</mn></msup><mo>+</mo><mi>A</mi><mo>*</mo><mi>C</mi><mo>+</mo><mi>B</mi><mi></mi><mi></mi></math><br></span>

Which is much simpler and very practical for entry. Future versions of the program may support formatting the equations this way for easy entry into calculators, likely through <span class="latex">L<sup>a</sup>T<sub>e</sub>X</span>.

# Conclusion
Whilst this program isn't very useful it's a fun little exploration of an idea, and could likely be pushed even further into full computations of complex programs. I think there are certainly limitations, which may be possible to overcome but I cannot determine that for certain. The whole ordeal feels like an obvious mathematical idea that I am simply unaware of (such as maybe being used for SAT Solving of logic circuits in hardware design) but I haven't found anything on cursory web searches. In the future if I can find a way to nicely resolve the simplification of equations a more structured parser in Haskell using parser combinators (or in Python with an equivalent library) might be pleasant, for example the above python program could be specified as:
```python
Sum=Xor(Xor(A,B), C)
Carry=And(And(C, Xor(A,B)), And(A,B))
```

The idea clearly isn't fully formed but it was nice to look back on an old curiosity and hopefully I can one day take it to its full conclusion, though that will likely need a good graph library for visual logic gate creation and a lot of proofs to find the full limits of the equivalence.