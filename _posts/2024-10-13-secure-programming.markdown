---
layout: post
title:  "Learning OCaml & Secure Programming Ideas"
date:   2024-10-13 00:19:13 +0100
categories: Ocaml Testing Security
---

# Introduction
This week I began looking into learning OCaml, with a focus on compiler design and creating secure computer programs.

# What Is OCaml?
OCaml is a functional programming language specialised in compiler design, used for the development of the [Coq theorem prover](https://coq.inria.fr/). It is not purely-functional, that is OCaml programs can easily introduce side effects, but are primarily written in a more functional style (maps, folds, pattern matches etc.).

To learn OCaml I followed the [CS 3110](https://cs3110.github.io/textbook/cover.html) textbook for the Cornell University course.

# Making Secure Programs: Asserts
One idea the textbook brought up was using language constructs to enforce the usage methods of a function, particularly through assert. For example a programmer looking at this function partially implementing the quadratic formula:
```python
def quadratic(a,b,c):
    return (-b + sqrt(b**2 - (4*a*c))) / (2 * a)
```

May not realise that for inputs where a = 0 the function will crash the program. In a stronger language with [dependent types](https://en.wikipedia.org/wiki/Dependent_type?useskin=vector) (such as OCaml with certain libraries) we could enforce this as a guarantee, preventing the function from producing an exception, or we could return a 'Maybe' type like Haskell. But in Python we don't have that, but we do have assert. We can make the function clearly express this limitation, raise an exception if it occurs and document exactly why it occurs all using assertions:

```python
def quadratic(a,b,c):
    assert a != 0, "'a' cannot be zero as it causes a division by zero"
    return (-b + sqrt(b**2 - (4*a*c))) / (2 * a)
```

We have described the API of the function more clearly, handled it cleanly when it is breached and even documented why the API is designed this way, making asserts a clean construct for clarifying programs and catching exceptional states in functions.

# Making Secure Programs: Fuzzing
I haven't yet tried fuzzing outside of OCaml so I will describe the process in QCheck, an OCaml fuzzing tool.

Here is a simple test to confirm that ```List.rev(List.rev(lst)) = lst```, that is, reversing a list twice gives the original list:
```ocaml
let test =
  QCheck.Test.make ~count:1000 ~name:"list_rev_is_involutive"
   QCheck.(list small_nat)
   (fun l -> List.rev (List.rev l) = l);;

(* we can check right now the property... *)
QCheck.Test.check_exn test;;
```

This code runs 1000 times, generating random lists of small integers and then checking the property. Fuzzing is interesting as it presents an alternative to simple testing that can better find weird edge cases in programs. Fuzzing's greatest weakness is that for programs with complex inputs, such as compilers, it may be hard to generate simple valid inputs, though [fuzzing has found compiler bugs before](http://www.vegardno.net/2018/06/compiler-fuzzing.html).

# Feelings About OCaml
Whilst I have talked about the textbook, and how OCaml itself has already taught me interesting ideas about making programs more secure I haven't spoken much about the language. OCaml is very pleasant, with its implicit typing (where the compiler finds the correct static types where possible) making writing clean, fast, functional code very easy. I also appreciate the less strict form of FP OCaml has compared to Haskell, making it feel more practically minded for writing complex, real-world programs. Overall I really like OCaml, though F#, the official Microsoft .NET derivative of OCaml, would likely be my choice for real-world programs due to the better library support and convenience of the .NET platform.

# Conclusion
In conclusion I am very happy with my time learning OCaml, I feel it has made me a slightly better programmer, with more of a security conscience and motivation to write cleaner programs that enforce concepts more explicitly (as opposed to just through comments). I might try writing an interpreter in OCaml soon as many people online say it is a good tool for the job, but otherwise I am unsure how much more I want to explore the language given its terseness and slight lack of practicality for general-purpose programming and solving real-world problems.