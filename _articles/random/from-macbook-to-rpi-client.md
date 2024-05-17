---
layout: page
title: My attempts to resurrect a 2008 MacBook as a VNC Client for a Raspberry Pi Server
published: false
---

## What you'll need
- A 2008 MacBook running Snow Leopard
- A Raspberry Pi 4 running Raspbian OS, with ssh enabled

## What you'll do
- On the MacBook:
    1. Install XCode from the Snow Leopard disc.
    1. Install MacPorts (You'll most likely need to download this on another machine then some physical media, like a thumb drive, to transfer it over).
    1. Using MacPorts, install the latest version of ssh.
    1. Test ssh connectivity to the RPi.
    1. Using MacPorts, install the latest version of `sudo port install tightvnc`
- On the RPi (via ssh, if you like):
    1. `sudo apt install tightvncserver`
    1. `vncserver :2 ... 1280x800` set password as part of flow
- On the MacBook:
    1. `vncviewer -fullscreen`
    1. Enter hostname including port, eg `raspberry-pi.local:5902` or `123.123.123.123:5902`
    1. If it works, congrats! If it doesn't, then sorry, but it worked for me.

## Further tweaks
    - VS Code keys went bonkers. Backspace didn't work, 5 was backspace, 6 was space. Fix this with: File > Preferences > Settings > Application > Keyboard > keyCode
    - Chromium froze, but only on certain pages. I didn't bother to investigate, I just switched to Firefox.
    - I'm sure that I'll find more bugs; if I find more fixes I'll add them here.

## Things I did and suggest you don't
    1. Use Snow Leopoard's built in VNC viewer
    1. Use RPi's built-in VNC server
    1. Attempt to use a Pi Zero 2 as the server
    1. Attempt to use a Windows machine as a server
    1. Attempt to use a Xubuntu machine as a server
    1. Attempt to use a RPi model B+ as the server
    1. Get ssh working by downgrading the hostkey algorithm
    