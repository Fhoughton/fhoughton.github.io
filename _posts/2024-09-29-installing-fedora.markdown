---
layout: post
title:  "Installing Fedora Linux"
date:   2024-09-29 00:19:13 +0100
categories: linux fedora
---

# Introduction
This week I was very busy with assignment and work but I managed to fit in some time to make my long-awaited switch from [Manjaro Linux](https://manjaro.org/) to [Fedora KDE](https://fedoraproject.org/spins/kde/), and I'll detail the benefits I've felt from this switch so far, my motivations and how I switched.

# What Is Fedora Linux?
Fedora Linux is the open-source form of Red Hat Enterprise Linux, a commercial Linux OS. It is somewhat seperate now, branching off slightly, but seeks to provide a good, stable, freedom (as in software) respecting Linux disribution.

# Why Switch?
I had a number of reasons for making the switch:
- Rolling release is great for developing but bad when you want things to just work every time, and I needed something stable for my final year of university
- I had heard great things about KDE, and having always been an XFCE user I wanted to give it a try 
- Lots of little things annoyed me or were broken on my Manjaro system, and I wanted something that didn't have bugs that got in my way (for example on Manjaro I could never quite fix the power manager sleeping my laptop after being left, something I dislike)
- I had a lot of cruft built up on my system, my hard drive was low on space despite few personal files and attempts to clear space weren't fruitful
- Fedora respects software freedom; Manjaro has non-free software in its repositories by default (e.g. Steam), Fedora has only free (as in open source) software by default, with opt-in proprietary packages (though it does ship firmware blobs, only GNU Guix doesn't really)

# Performing The Switch
To perform the switch I first backed up all of my files, copying them to a drive and then flashed the iso for Fedora KDE using the official Fedora Media Writer, which is a lovely addition to the install, ensuring a correct and easy way to flash the operating system to a USB stick (over Rufus or Balena Etcher, which can break certain systems and kill USBs). The Fedora Media Writer also happily restores your USB stick after use, preventing breakages due to messed up partition tables.

# Benefits I gained
- More consistent package naming, if I needed headers for some package packagename I know they're always in packagename-devel (for example ruby-devel for setting up jekyll on the new system)
- Things just work, Bluetooth was broken on my Manjaro install and now just works out the box on Fedora
- More configurability, I can change the window themes, tweak the sound levels by minute percentage and just customise all of the system thanks to the design behind Fedora and KDE
- Fedora just does things better, it's like if Windows was made for Linux and actually respected the user
- Great innovations that just speed up the day-to-day for example when setting back up Jekyll locally I executed ```bundle exec``` in my terminal, got command not found and Fedora offered to install the packages I needed for me, letting me get straight past finding the right packages and straight into using Jekyll
- Dnf is just lovely, it's so clear in its usage and just works great, for example in Manjaro to install a package you do ```sudo pacman -S packagename``` in Fedora it's just ```sudo dnf install packagename```, much clearer and easier to remember, as well as making commands you don't know much more guessable (which is how I learned ```dnf remove```)

# The Little Annoyances
Fedora isn't perfect, some things do bug me still:
- The toolbar is way too big and the window titlebars are too large, shrinking windows
- The audio mixer can't display my sound settings well and struggles with Electron apps due to them disposing of their audio handle often
- The way to switch file open defaults copies windows, making setting a program as the default to open a file annoying

But these are minor and the toolbar and titlebar issues only persist because shrinking them looked odd, not because the option was missing.

# Conclusion
Overall I'm very glad that I switched to Fedora KDE, I love the KDE gui (I tried GNOME before and hated it) and it feels like KDE wants to work to help me get what I need to do done. Rpms are easy to install, the package manager is fantastic and it's just a nice, well-built system. It feels like when switching between Windows and Linux you always have to compromise in some way, Fedora minimizes much of the Linux bugginess that comes with the Linux side of the compromise, making it a great system with the polish of Windows and the freedom and power of Linux. I feel that Fedora is nearly the perfect operating system to me, with my only gripes being small usability issues, its corporate ties and the inability to run all Windows programs (as all Linux distros suffer from).
