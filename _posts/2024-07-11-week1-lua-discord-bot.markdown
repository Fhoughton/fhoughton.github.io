---
layout: post
title:  "APAW Week 1: A Discord Bot In Lua"
date:   2024-07-11 00:19:13 +0100
categories: lua discord api
---

This the first of a series of projects/posts I'm going to try writing up called 'A Project A Week' (APAW), where I try to do a smaller project each week to learn new skills and create useful tools. The scope of the projects might vary based on the amount of things in the week but the goal is to try and learn *something* new each week.

The repo for this project can be found [here](https://github.com/Fhoughton/LuaDiscordBot).

# Why This Project?
I have been wanting to learn [Lua](https://www.lua.org/) for a while, as it pairs very well as a scripting language for C and C++ based applications, with its ease of embedding and light weight. Lua sports other advantages too, such as its ability to be sandboxed and the great speed it can reach using [LuaJIT](http://luajit.org/) compared to embedding alternatives such as Python (which suffers greatly due to the [GIL](https://realpython.com/python-gil/) too. though there's [work to fix this](https://peps.python.org/pep-0703/)).

As I was looking for projects to use as an avenue to learn Lua I came across a small programming competition for making bots on the chat platform [Discord](https://discord.com/). The bots had to be puzzle themed, and so my project was born.

Lua is a nonstandard language for this field, with Javascript and Rust usually being preferred, but Lua didn't create any issues and was pleasant to work with throughout the development period.

# What Can The Bot Do?
The final bot was produced using [Luvit](https://luvit.io/), a Lua fork with asynchronous I/O. It was capable of:
- Per-user save files, allowing multiple users
- 5 commands to interact with the bot/game state
- Prefix (bot command identifier) switching

The puzzle game I designed aimed to teach simple [Reverse Polish Notation (RPN)](https://en.wikipedia.org/wiki/reverse_Polish_notation), a form of mathemtical expression that removes the needs for brackets in deciding order. For example ```2 + (3 * 4)``` would become ```3 4 * 2 +```, eliminating brackets. This notation is extremely simple to parse and implement since you can process it by pushing numbers to a stack and then applying the equations to the stack. This processing method is used in the embedded programming language [Forth](https://en.wikipedia.org/wiki/Forth_(programming_language)), which I'm a big fan of.

# Running The Bot
When the bot is [installed and run](https://github.com/Fhoughton/LuaDiscordBot/blob/master/README.md), with a bot account and token tied to it, the bot may be invited to a channel where it can be interacted with to start the puzzle game. The game had 7 levels and was started by executing ```!about``` which explains the story and gameplay of the puzzle game.

![](/images/luabotabout.png)

Once the game is started the user can send moves to work through the levels:

![](/images/luabotusage.png)


# What I Learned
The project taught me a lot about the language design and philosophy of Lua. The language is great for 'glue' code, which joins large chunks programmed in a more complex language. The simplicity of the language, with just 20 keywords (compared to C++'s 95), and just one complex data structure with tables (which are associative arrays), makes the language possible to hold easily in your head, reducing friction in making design decisions.

The Discord API was also quite intuitive and its architecture confirmed my assumptions about how such APIs should be designed, as well as how Discord itself would be designed (for example each user has a numeric UUID despite also having unique usernames, for the purpose of consistency on username changes and optimisation).

I also learned just how fast a JIT can be, as I had previous knowledge about PyPy but LuaJIT is in a league of its own, only being 10% slower than handwritten equivalent C code in [some benchmarks](https://old.reddit.com/r/lua/comments/htqn0t/luajit_once_again_nearly_as_fast_as_the/), despite being dynamically typed and very simplistic.

Overall the project was quite cathartic, helping me learn Lua nicely and confirm my intuitions about certain application design ideas. Lua seems to me to be a pleasant language, with great features for embedding in C++ codebase where Python might cause issue, and is such a simple and intuitive language that it causes little mental strain to work in relative to more complex languages like Rust. For this reason Lua has a place as possibly one of my favourite languages to program in, especially as it is extremely performant, with LuaJIT having [20 times the speed of Python](https://reddit.com/r/Compilers/comments/llujxv/luajit_performance/) in benchmarks!