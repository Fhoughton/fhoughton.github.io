---
layout: post
title:  "APAW Week 7: Experimenting With Prolog"
date:   2024-08-24 00:19:13 +0100
categories: prolog
---

This week I experimented with the logic programming language [Prolog](https://en.wikipedia.org/wiki/Prolog), a weird and slightly whimsical programming language unlike many others. I constructed some basic programs along the way and got to experiment with constraint programming, leading to some *cool* results.

In this article I will provide my implementations for:
- A basic family database
- A basic tree classifier
- A system for managing customer warehouse orders
- [Collatz Conjecture](https://en.wikipedia.org/wiki/Collatz_conjecture) exploration

# Predicates And Parents
I had some previous experience in Prolog, but hadn't cemented anything beyond the basics with programs, so I decided to work through [Adventure In Prolog](http://www.amzi.com/AdventureInProlog/index.php), a book on implementing a text adventure game in Prolog. As I went along I became more interested in the smaller learning tasks, so I worked towards them and branched out to my own projects with the techniques the book taught me.

But first I began by following the guide in the book for a basic geneology program:
```prolog
% Define the individuals
male(dennis).
male(michael).
female(diana).

% Define their relationships
parent(dennis, michael).
parent(dennis, diana).

% Provide a rule for finding out if an individual is someone's mother
mother(M,C):-
  parent(M,C),
  female(M).

% Provie a rule for finding is someone is a sibling
sibling(X,Y) :- parent(Z, X), parent(Z, Y), X \= Y.
```

With just these definitions we can make some interesting reasonings about the data we have described, for example we can find all parents:
```prolog
?- parent(X, _).
X = dennis ;
X = dennis.
```

But also all children, using the same definition:
```prolog
?- parent(_, X).
X = michael ;
X = diana.
```

Or find all siblings:
```prolog
?- sibling(X,Y).
X = michael,
Y = diana ;
X = diana,
Y = michael ;
false.
```

Notice that prolog returns false when it is done searching, this is because to resolve a query it attempts to satisfy all the given constraints, and then backtracks to solve every other path upon completion, and when it eventually runs out of possible paths it concludes that the remaining query is false, as there are no more possible forms to satisfy.

# A Tiny Tree Identifier
I once read the book [*How to Read a Tree: Clues and Patterns from Bark to Leaves*](https://www.amazon.co.uk/How-Read-Tree-Patterns-Navigation/dp/1615199438) by Tristan Gooley, a great book all about the intricate details about the environment and themselves trees will reveal (a fun one is that trees can't heal, only grow, so when infected or damaged they kill the part of them that is the problem, which means that large bulbs in a trees trunk are where it has isolated an infection then grown bark around it). The very start of the book defined the basics of how you can roughly spot if a tree is evergreen or not, and I very roighly encoded some of these ideas into a small Prolog program:
```prolog
coniferous(X) :- leaves(X, pointy), evergreen(X, true).
broadleaf(X) :- leaves(X, broad), evergreen(X, false).

leaves(oak, broad).
evergreen(oak, false).
```
This program states that coniferous trees are evergeen and have pointy leaves, whilst broadleaf (deciduous) trees have broader leaves and aren't evergeen (they shed their leaves in the winter). It then describes an oak tree, with facts that allow it to be deduced as a broadleaf tree. Whilst this program is a gross oversimplification of the truth it presents some of the power a knowledge database can bring, for example if a more complete list of trees and their traits were found a system could be constructed to identify trees by letting the user enter details about the tree, or a programmer could pose queries to find what trees satisfy some constraints they arbitrarily decide (such as if they really wanted a short evergreen tree for their garden).

# A Wider Warehouse System
Next I decided to construct a much larger program, with some guidance from the adventure game book. I chose to construct a warehouse system, which could fulfil orders and answer queries about its stock. The listing is below:
```prolog
% custom/3(name,city,credit [aaa,bbb, etc.])
customer(steve,london,aaa).

% item/3(id,name,reorder[when at or below reorder])
item(0,screwdriver,5).
item(1,stapler,3).

% describes the stock left for an item by id e.g. inventory(0,7). means item id 0 has 7 items left in stock
:- dynamic inventory/2.
inventory(0, 7).
inventory(1, 2).

% gets the item quantity by name
item_quantity(X,Y) :- item(_,X,Y).

% prints a detailed report for an item and its quantity
% to get all item quantities do '?- inventory_report(X,Y).'
inventory_report(X,Y) :-
    item_quantity(X,Y),
    write(X),
    tab(1),
    write(Y),
    nl,
    fail.

% checks if an order of item Y, from person X can be fulfilled (there is enough stock to fill it)
valid_order(X, Y, Z) :-
    item_quantity(Y, Q),
    Q >= Z.

% prints if any items have to be reordered because they are below their threshold
reorder(X) :-
    item(I,N,R),
    inventory(I,C),
    C =< R,
    write('Reorder'),
    tab(1),
    write(N),
    nl,
    fail.

% sets an inventory item (by id) to a new stock quantity
update_inventory(X, Y) :-
    retract(inventory(X,_)),
    asserta(inventory(X,Y)).

% fulfils an order, changing the stock and adding it to the history
% as well as telling the user if items need to be reordered
order :-
    write('Enter customer name:'),
    read(C),
    nl,
    write('Enter item name:'),
    read(I),
    write('Enter the item quantity:'),
    read(Q),
    valid_order(C,I,Q),
    write('Processing order...'),
    nl,
    asserta(order(C,I,Q)),
    item(R,I,Z),
    write('Updating inventory...'),
    nl,
    update_inventory(R,Z-Q),
    reorder(X),
    fail.
```

And here is an example interactive session:
```prolog
?- inventory_report(X,Y).
screwdriver 5
stapler 3
false.

?- valid_order(steve, screwdriver, 10).
false.

?- valid_order(steve, screwdriver, 4).
true.

?- order.
%    Enter customer name:steve.
%
%    Enter item name:|: screwdriver.
%    Enter the item quantity:|: 5.
%    Processing order...
%    Updating inventory...
%    Reorder screwdriver
%    Reorder stapler
     false.
```
As you can see with this program:
- The stock can be queried
- Orders can be made and rejected as needed
- Interactive user menu can be presented that:
    - Allows users to place orders
    - Checks if the order can be fulfilled
    - Updates the inventory in response to the order
    - Tells the user what needs to be reordered after the changes have been make.

# Constraints And Collatz
Constraint programming is a cool extension of logic programming that enables reasoning over integers. For example you can request that only integers that are less than 12 are returned from a predicate. Using this I constructed a program to reason about the Collatz Conjecture which is as follows:
- If a number is even half it
- If a number is off triple it and add 1
- This sequence will always reach the number 1 (for any postive integer)

The program I wrote is as follows:
```prolog
:- use_module(library(clpfd)).

% Gets the next sequence in the collatz conjecture for a given integer N0
collatz_next(N0,N) :- N0 #= 2*N.
collatz_next(N0,N) :- N0 #= 2*K + 1, N #= N0*3 + 1.

% Checks if you can reach from some integer N0 the number N at some point in the sequence
collatz_reachable(N, N).
collatz_reachable(N0, N) :- collatz_next(N0, N1), collatz_reachable(N1, N).

% Check if some starting number gets above some goal integer G
collatz_above(S, G) :- N #> G, collatz_reachable(S, N), !.

% Test the collatz conjecture for some integer N0
collatz_test(N0) :-
    \+ \+ collatz_reachable(N0, 1). % \+ \+ is not not, makes it into a constraint to prevent looping
```

And we can use this program to reason about the conjecture like so:
```prolog
% Find the next number in the sequence for 2 (2 / 2)
?- collatz_next(2, X).
X = 1 ;
false.

% Find the next number in the sequence for 3 (3*3 + 1)
?- collatz_next(3, X).
X = 10.

% Find the next number in the sequence for 10
?- collatz_next(10, X).
X = 5 ;
false.

% Since the sequence for 3 goes 3 -> 10 -> 5, we can query if 3 reaches 5 at any point:
?- collatz_reachable(3, 5).
true.

% We can check if from a given starting number it goes above a certain value in the sequence
?- collatz_above(10, 15).
true.

% We can check the conjecture is satisfied for a given number
?- collatz_test(10).
true.

% And we can test for all integers if the constraints are satisfied (which they are as the conjecture claims)
?- length([_|_], N0), collatz_test(N0).
N0 = 1 ;
N0 = 2 ;
N0 = 3 ;
N0 = 4 ;
N0 = 5 ;
N0 = 6 ;
N0 = 7 ;
N0 = 8 ;
N0 = 9 ;
N0 = 10 ;
N0 = 11 ;
N0 = 12 ;
N0 = 13 ;
N0 = 14 ;
N0 = 15 ;
...
```

# A Cheery Conclusion

***Prolog is fun!***

I greatly enjoyed working with Prolog. It provides an interesting alternative take on programming, replacing sets of instructions with descriptions of worlds and the rules that govern them. It is calming in a way, and it is very satisfying when a program works well, and you can use a basic definition to reason about much more complex things, and to solve the inverse of the problem. These skills are mythical in the language, with a famous example being that if you can solve a given puzzle (prologs specialty) you can also generate new puzzles following certain constraints, such as creating Sudoku boards of a certain size.

***Prolog is weird!***

Prolog is very weird, but in a good way mostly. It can do difficult things easily, and easy things difficultly. It is certainly the best tool for certain jobs, particularly for solving puzzles or making expert systems, but it is hopeless at more linear tasks, such as extracting files or performing fast calculations. Prolog has very few libraries and its slightly weird syntax makes it more obtuse than it needs to be (it's owing to its history of coming from the concept of [Horn Clauses](https://en.wikipedia.org/wiki/Horn_clause)). But I think overall, in spite of these quirks, I love it a lot and it wouldn't be the same without them. Prolog is weird, and that's also why it's fun.

Overall I think I can summarise my thoughts on Prolog as such:
*I really like Prolog, even if it isn't the most useful programming language in the world, because it is just great fun to work with*