---
layout: page
title: My attempts to resurrect a 2008 MacBook as a VNC Client for a Raspberry Pi Server
---

## What you'll need
- A 2008 MacBook running Snow Leopard
- A Raspberry Pi 4 running Raspbian OS, with ssh enabled

## What you'll do
- On the MacBook:
    1. Install XCode from the Snow Leopard disc.
    1. Install [MacPorts](https://www.macports.org/) (You'll most likely need to download this on another machine then use some physical media, like a thumb drive, to transfer it over).
    1. Using MacPorts, install the latest version of ssh: `sudo port install ssh`.
    1. Test ssh connectivity to the RPi: `ssh pi@raspberry-pi.local`.
    1. Using MacPorts, install the latest version of tightvnc: `sudo port install tightvnc`
- On the RPi (via ssh, if you like):
    1. `sudo apt install tightvncserver`
    1. `vncserver :2 -geometry 1280x800 -depth 24` and set the vnc password as part of flow
- On the MacBook:
    1. `vncviewer -fullscreen`
    1. Enter hostname including port, eg `raspberry-pi.local:5902` or `123.123.123.123:5902`
    1. If it works, congrats! If it doesn't, then sorry, but it worked for me.
    2. Make connecting easier by running `vncpasswd` and storing the vnc password on the MacBook.
    3. Connect using the stored password file: `vncviewer -fullscreen -... `

## Further tweaks
- VS Code keys went bonkers. Backspace didn't work, 5 was backspace, 6 was space.
    - Fix this with: File > Preferences > Settings > Application > Keyboard > keyCode
- Chromium froze, but only on certain pages. I didn't bother to investigate, I just switched to Firefox.
- I'm sure that I'll find more bugs; if I find more fixes I'll add them here.

## Things that I tried and suggest that you don't
1. Use Snow Leopoard's built in VNC viewer
1. Use RPi's built-in VNC server
1. Attempt to use a Pi Zero 2 as the server
1. Attempt to use a Windows machine as a server
1. Attempt to use a Xubuntu machine as a server
1. Attempt to use a RPi model B+ as the server
1. Get ssh working by downgrading the hostkey algorithm

The first six I tried and they all failed, for one reason or another. The seventh worked, but is shonky; upgrading ssh to use current hostkey algorithms is a much better approach.
