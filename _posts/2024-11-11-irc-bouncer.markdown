---
layout: post
title:  "Running An IRC Bouncer"
description: Setting up an IRC bouncer, a bot that sits in channels and stores their messages to keep a text history
date:   2024-11-11 00:19:13 +0100
categories: IRC Raspberry-Pi
---

# What Is IRC?
IRC (internet relay chat) is a chat service where clients communicate via a chat server (irc.libera.chat in my case) and join channels. Connections are made via TCP and optionally with SSL (which I will be using in the post for security).

# What Is An IRC Bouncer?
An IRC bouncer is software that hangs in IRC channels, recording all messages. This allows chat message history to be viewed as IRC does not store chat messages after they are posted, and they are only visible to users of the channel (which is seen by some as a positive). I personally need one so that I can catch discussions when I'm away in hobbyist channels such as GNU Guix's #guix channel.

# Setting Up A Raspberry PI Zero

<img src="/images/irc-board.jpg" width=600>

I began by setting up a raspberry pi zero, as it has low power draw and runs linux, making it an easy choice for a low-power client to run the bouncer. The Linux support enabled me to use ZNC, the most famous IRC bouncer software.

I began by flashing the raspbian image to the microSD card:

<img src="/images/irc-raspi-imager.png" width=500>

<img src="/images/irc-raspi-doneimager.png" width=500>

Then I connected the device via microHDMI and powered it from the USB port of my PC:

<img src="/images/irc-board-on.jpg" width=300>

# Configuring ZNC
I logged on, installed and setup ZNC (forgive the blurred images, I'm hiding the username and will alter the hostname as the device will be public):

<img src="/images/irc-raspbian-config.jpg" width=600>
<img src="/images/irc-znc-install.jpg" width=600>
<img src="/images/irc-znc-config.jpg" width=600>

Then I created systemd service and let it run:

<img src="/images/irc-znc-systemd.jpg" width=600>

<img src="/images/irc-znc-systemd-status.jpg" width=600>

# The Result
Finally I could view the bouncer and its history:

<img src="/images/irc-znc-commands.png" width=800>

# Conclusion
Setting up an IRC bouncer was fairly quick and will hopefully prove very useful into the future. It seems to be a must nowadays, as IRC is very slow moving and importanting topics can get lost on it for niche software. I'll have to leave it running and see what happens but it should hopefully be very helpful for me as I move towards using IRC more.