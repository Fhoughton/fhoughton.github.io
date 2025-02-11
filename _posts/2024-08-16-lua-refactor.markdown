---
layout: post
title:  "Doom Emacs And Refactoring A Lua Codebase"
description: Trying out Doom Emacs, a modern pre-configure emacs, and using it when refactoring some Lua code for a game
date:   2024-08-18 00:19:13 +0100
categories: Lua Refactoring Emacs
---

This week I was fairly tight for time but I spent the weekend refactoring a lua codebase I had for my game engine in preparation for a game jam, and I gave [Doom Emacs](https://github.com/doomemacs/doomemacs) a try.

<img src="/images/doom-logo.png" width="200"/>

<small> <i> The Doom Emacs Logo </i> </small>

# Why Refactor The Code?
My choice to refactor the codebase came from the realization that having not opened it for around a month it seemed to be a complete mess. It was filled with lots of complex libraries that were being used throughout, it had lots of code relating to shared features (such as scaling) split across files, and it had a number of terrible design decisions.

# Why Doom Emacs?
I was interested in trying Doom Emacs as an alternative to [Visual Studio Code](https://code.visualstudio.com/) for a number of reasons:
- It is faster and more lightweight than VSCode, reducing the burden on my laptop
- Doom has better support for nice languages (such as [Agda](https://agda.readthedocs.io/en/latest/getting-started/what-is-agda.html)) than VSCode
- Doom is easier to configure, and makes trying new languages fast
- 'Org Mode' for editing documents
- Keyboard-driven development
- Doom is much more featureful in its configuration, making control of certain features much easier

# Refactoring Technique 1: Extracting Functions
The most simple refactoring technique is to break up huge functions into seperate subroutines, for example I moved my player movement code to a seperate function, simplifying the main loop:

**Before**

<small> <i> The Old Main Loop </i> </small>

```lua
function Player:update(dt)
  self.image:update(dt)
  self.image:play()

  local deltaX, deltaY = 0, 0

  if love.keyboard.isDown("left") then
    deltaX = -self.speed * dt
    self.image:setTag("WalkLeft")
  elseif love.keyboard.isDown("right") then
    deltaX = self.speed * dt
    self.image:setTag("WalkRight")
  elseif love.keyboard.isDown("up") then
    deltaY = -self.speed * dt
    self.image:setTag("WalkUp")
  elseif love.keyboard.isDown("down") then
    deltaY = self.speed * dt
    self.image:setTag("WalkDown")
  else
    self.image:pause()
  end


  -- Calculate new position based on input
  local newX = self.x + deltaX
  local newY = self.y + deltaY

  -- Resolve collision
  local actualX, actualY, cols, len = collision_world:move(self, newX, newY, self.collision_filter)

  self.x = actualX
  self.y = actualY

  -- Camera follows the player's new position
  local middle = self:getMiddle()
  camera:lookAt(middle.x, middle.y)
end
```

**After**

<small> <i> The Extracted Movement Code </i> </small>

```lua
function Player:handle_movement(dt)
  local deltaX, deltaY = 0, 0

  if love.keyboard.isDown("left") then
    deltaX = -self.speed * dt
    self.image:setTag("WalkLeft")
  elseif love.keyboard.isDown("right") then
    deltaX = self.speed * dt
    self.image:setTag("WalkRight")
  elseif love.keyboard.isDown("up") then
    deltaY = -self.speed * dt
    self.image:setTag("WalkUp")
  elseif love.keyboard.isDown("down") then
    deltaY = self.speed * dt
    self.image:setTag("WalkDown")
  else
    self.image:pause()
  end

  return deltaX, deltaY
end
```


<small> <i> The New Main Loop </i> </small>

```lua
function Player:update(dt)
  self.image:update(dt)
  self.image:play()

  local deltaX, deltaY = self:handle_movement(dt)

  -- Calculate new position based on input
  local newX = self.x + deltaX
  local newY = self.y + deltaY

  -- Resolve collision
  local actualX, actualY, cols, len = collision_world:move(self, newX, newY, self.collision_filter)

  self.x = actualX
  self.y = actualY

  -- Camera follows the player's new position
  local middle = self:getMiddle()
  camera:lookAt(middle.x, middle.y)
end
```

As you can see, the code is much clearer, and the functionality is much more obvious. The main loop functions in much more defined, smaller stages and the movement function is very linear and obvious. This extraction also allows for better testing of smaller pieces of code, which is facilitated through the use of the lua console.

# Refactoring Technique 2: Reorganising File Locations

Where a file lies in the codebase is just as important as its name and structure. In my codebase I had game states (gameplay, title etc.) and the objects tied to the states were included with them, in their own folders. Inspired by the file structure of [GameMaker](https://gamemaker.io/en) I moved all of the active game objects to one folder called 'Objects'.

# Refactoring Technique 3: Spotting Dead Code

<img src="/images/refactor-delta.png" width="200"/>

When refactoring it is often obvious that some code was added on a false assumption, and can be safely removed to simplify the codebase further. After moving the objects used by game states to their own folder I completely removed game states from the game, instead opting for rooms filled with objects, as the two mechanisms already solved the issue that states aimed to. This let me remove the game state library altogether, removing hundreds of lines of hidden code and letting me clear up lots of dead files tied to states I wasn't using.

Furthermore as files are moved and audited outdated code from vestigal features often jumps out and can safely be removed:

![](/images/refactor-unused.png)

In this case I found some unused debugging code (inspect.lua) and was able to simplify the imports to files due to the folder reorganising.

# Refactoring Technique 4: Combining Modules

Sometimes two modules are made that achieve similar aims in different contexts. In my codebase I found I had a helper file for hooking into VSCode's lua debugger (lldebugger) named 'debug.lua' and a file for debugging gameplay code called 'gameplay_debug.lua'. 

I merged the two files together in the refactor that removed scenes, reducing the total number of files in the repository and keeping similar code together, making it more obvious where to go to find any debug code. 

# Conclusion

**Refactoring**

The refactoring was a success, simplifying the codebase and removing hundreds of lines in the process. Code was merged and extracted to more logical modules and functions were extracted to simplify complex logic and create a more functional design patter, with better support for testing. 

![](/images/refactor-change.png)

<small> <i> The End Result, With A Lot Of Code Shaken </i> </small>

By the end I felt confident that I once again understood the codebase and was no longer overwhelmed by the complexity of having multiple ways to do the same thing. I think there are large considerations to be learned from the whole process, and that perhaps my next engine attempt will learn from them and be much cleaner than those that came before, but only time will tell.

**Doom Emacs**

<img src="/images/doom-yay.png" width="200"/>

<small> <i> Me After Using Doom Emacs </i> </small>

<small> <i> (Evil Is The Vim Binding For Emacs) </i> </small>

I was extremely satisfied with Doom Emacs. It was consistently snappy, easy to configure, and responded well to whatever I needed it to do. I definitely still have growing pains, such as with compiling and running the codebase *(SPC c c)* from inside of folders other than the root folder (Edit: turns out the solution is *projectile-command-file (SPC p c)*), and remembering complex keybinds. However, I was very happy with my time spent with Doom overall and I will continue to try and use it in place of VSCode, and I will hopefully iron out all the pains and have an even better, faster code editing experience.
