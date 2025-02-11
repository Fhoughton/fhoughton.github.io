---
layout: post
title:  "Trying Org Agenda And Prolog DCGs"
description: Trying the Emacs org-agenda package for organising a calendar, and using DCGs, a special parsing technique, in Prolog
date:   2024-09-08 00:19:13 +0100
categories: Prolog DCG Emacs Org-Mode
---

# A Slightly Mixed Week
This week I spent most of my time learning new technologies so it's a bit of a mixed bag, I'll try and go over what I learned and how you can practice it too, as well as my key learnings. The focus of this post will be on Declarative Clause Grammars (DCGs) in Prolog, a high-level literate parsing technique, and org-agenda, an emacs package that helps with creating schedules.

# Declarative Clause Grammars
Declarative Clause Grammars (DCGs) are a declarative way of defining grammars and their parsers (as well as the inverse, being able to produce texts from parse trees). 

```prolog
sentence --> [b],[a],[b].
sentence --> [b], sentence, [b].
```

The code above defines a language that accepts any phrase with an equal number of b's and an a in the middle, so the text 'bbabb' is valid but 'babb' is not. You'll notice that this descriptions is fairly high level, for example the equivalent in bnf would be:
```bnf
<sentence> ::= "bab"
<sentence> ::= "b" <sentence> "b"
```

DCGs can also bind to parsed content, providing a pleasant way to parse and extract information from textual content.
```prolog
like(What) --> "I like ", list(What), ".", list(_).

list([]) --> [].
list([L|Ls]) --> [L], list(Ls).
```
This would be able to parse "I like cheese." and bind "What" to "cheese", allowing us to act upon the parsed content easily.

DCGs are quite flexible, even being the way [prolog implements C code ffi](https://github.com/JanWielemaker/ffi) (python too!)

They do have some issues, such as with error messages, but they are a continual source of research and ideas are being worked on to fix this (a similar problem to parser generators before them).

# Org-Agenda
[org-agenda](https://orgmode.org/manual/Agenda-Views.html) is an emacs package based on the org-mode package, a literate document format. For example a heading with a list of the basic keybindings for org-agenda would be:
```org
* Usage
- SPC-m-t-t to set state
- C-RET to make new line/list item
- SPC-m-d-s to set date for a task
- SPC-o-a-a for agenda commands
```

With org-agenda files are used to keep track of tasks to do, as well as their time and other information, for example:
```org
** TODO Complete APAW Project [1/6]
SCHEDULED: <2024-09-08 Sun +1w>
+ [X] Learn org-agenda
+ [ ] Aur Helper
+ [ ] Doom Emacs Module
+ [ ] .net bytecode modifier/viewer (maybe JVM instead due to my Java usage with android)
+ [ ] Red-black tree
+ [ ] Raycaster game
```
This specifies a singular task, set as a todo, which is to complete the APAW project for each week. The "+ [ ]" act as checkboxes in a list, and each can be toggle on and off with space, with the amount left to do automatically updating, and the task completing when all are checked off. You can also manually complete the task with space.

The field ```SCHEDULED: <2024-09-08 Sun +1w>``` states the day that the task is scheduled for completion, as well as how often it will repeat (+1w means once a week). When the task is completed it only becomes completed for that week and is automatically retracked as to do during the next week.
```org
SCHEDULED: <2024-09-29 Sun +1w>
:PROPERTIES:
:LAST_REPEAT: [2024-09-09 Mon 02:58]
:END:
- State "DONE"       from "TODO"       [2024-09-08 Wed 12:58]
- State "DONE"       from "TODO"       [2024-09-15 Sun 09:42]
```

The agenda can be filtered, searched, viewed and filtered by tag (e.g. TODO) making it easy to manage lots of calendar events of different types and to track down information on when things are scheduled etc. To access the agenda view in Doom Emacs you use the keybinding SPC-o-A (Space then o then shift+a), from which you can select what you want to see in the agenda. 

For example the week view can be selected by pressing 'a':
```org
10 days-agenda (W36-W37):
Friday      6 September 2024
Saturday    7 September 2024
Sunday      8 September 2024
  agenda:     21:30 ┄┄┄┄┄ Scheduled:  TODO Complete APAW Project [0/10]
Monday      9 September 2024 W37
Tuesday    10 September 2024
Wednesday  11 September 2024
Thursday   12 September 2024
Friday     13 September 2024
Saturday   14 September 2024
Sunday     15 September 2024
```

Or the todo list by pressing 't':
```org
Global list of TODO items of type: ALL
Press ‘N r’ (e.g. ‘0 r’) to search again: (0)[ALL] (1)NO (2)YES (3)OKAY (4)[X] (5)[?] (6)[-] (7)[ ] (8)KILL (9)DONE (10)IDEA (11)HOLD (12)WAIT (13)STRT (14)LOOP (15)PROJ (16)TODO
  agenda:     TODO Complete APAW Project [0/10]
  agenda:     TODO Learn FamiStudio [2/10]
```

Org-agenda is a good way to keep a calendar, but it also helps with direct task tracking through its support of the org-mode format, with things such a bulleted lists. It's a small but powerful improvement on a traditional calendar, combining it with a todo list and allowing much more effective querying and more dynamic usage (for example it can also be used to store a list of ideas). Org-agenda is a highly flexible tool and I think it has greatly helped me organise my planning and thoughts already.

Org-agenda isn't flawless, in particular getting it integrated with my Google Calendar smoothly seems to be painful, and it's something that I want to follow up on more as I have to frequently view it for work, and having everything in one place would make everything easier. I will explore this next week.

# Conclusion
Whilst no grand project was completed this week I feel I learned a valuable new technology in DCGs, and that learning org-agenda has set me up to be more efficient and organised for the forseeable future (and so was of extreme value to learn).
