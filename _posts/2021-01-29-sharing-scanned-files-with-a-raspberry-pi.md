---
layout: post
title:  "Sharing scanned files with a Raspberry Pi"
---

With multiple children on multiple devices, and just one scanner, I recently set up a Raspberry Pi to share scanned files across my home network.

Rather than start by telling the story of how I got here, I'll give a the briefest possible description of the final, working solution first, so that anyone copying this can get straight to the good bit, and then we'll delve into all the quirks and failures at the end.

# Equipment
My setup consists of:
- A Canon PIXMA MG5350 multi-functional (scanning & printing) device.
- A Raspberry Pi 2 Model B, running Raspberry Pi OS.
- Several client machines, including a Chromebook and laptops running Ubuntu.

# How to blindly copy what I've done, in as few words as possible

Keep an SD card permanently inserted into the MG5350 SD card slot; when scanning, save images to this SD card.

Connect the MG5350 via USB to the Raspberry Pi.

On the RPi, install usbmount using:
{% highlight sh %}
wget https://github.com/nicokaiser/usbmount/releases/download/0.0.24/usbmount_0.0.24_all.deb
dpkg -i usbmount_0.0.24_all.deb
{% endhighlight %}

Save the following script as `/home/pi/scanner-sync.sh` and run `chmod +x /home/pi/scanner-sync.sh` 
(In reality, this script is highly unlikely to work as-is for you, but we'll get into that later)
{% highlight bash %}
#!/bin/bash
set -e
if [[ $EUID -ne 0 ]]; then
   echo "This script must be run as root" 
   exit 1
fi
source_dir=/media/usb0/CANON_SC/
destin_dir=/home/pi/max/scanner/
if [[ ! -d $source_dir ]]
then
 echo "$source_dir is not an existing directory"
 echo "Scanner is probably off"
 exit 1
fi
if [[ ! -d $destin_dir ]]
then
 echo "$destin_dir is not an existing directory"
 echo "NAS is probably off"
 exit 1
fi
echo "Initial sync of files from scanner to NAS"
rsync -a --no-g --no-o --update $source_dir $destin_dir
echo "Reloading media card to pick up new files"
echo "Removing..."
udevadm trigger --action=remove /dev/sda1
sleep 10
echo "Re-adding..."
udevadm trigger --action=add /dev/sda1
sleep 10
echo "Re-syncing files from scanner to NAS, to pick up new changes"
rsync -a --no-g --no-o --update $source_dir $destin_dir
echo "Done"
exit 0
{% endhighlight %}

Configure cron to run the script every minute as root, by running `sudo crontab -e` and adding the following line at the end:

`* * * * * /home/pi/scanner-sync.sh 2>&1 | /usr/bin/logger -t scanner_sync`

Finally, share the `/home/pi/max/scanner` directory as a scanner folder by adding the following to the bottom of `/etc/samba/smb.conf` for a previously-configured Samba setup:

{% highlight sh %}
[scanner]
path = /home/pi/max/scanner
writeable=no
public=no
browseable=yes
{% endhighlight %}

Now, all of your clients, including your Chromebook, can connect to the SMB `/scanner` share and download the scanned files without fighting over the memory card.

# More words, more details
I appreciate that such a brief overview won't be enough for everyone, as it certainly wouldn't have been enough for me. If you'd like to know more, or if you've blindly copied all that and it hasn't worked, read on.

## The scanner

The Canon PIXMA MG5350 is an all-in-one device that scans, prints and copies, and along with USB connectivity it has Wifi connectivity and a memory card built-in.

For my situation, I don't need to use the printer at all (we already have another network printer for that), and I don't need to scan from the RPi itself either, so if you're looking for a tutorial on setting up an MG5350 to be used from a linux machine then, sorry, this isn't it.

Along with the usual print and scan functionality, the MG5350 includes the capability to share the contents of an inserted SD card either a) over the Wifi as an SMB share, or b) via the USB cable as a USB drive.

An SMB share over Wifi sounds perfect; all of the clients (even the Chromebook) support accessing SMB shares, so let's just do that and not bother with the RPi at all! The only problem is, the MG5350 is something of an aged beast, and its SMB support is stuck at v1, whereas the Chromebook only supports SMB v2 and above. Sadly, this is never going to work.

Initially, I did try accessing the MG5350 via SMB v1 from the RPi; I already had experience of this, having connected the RPI to an aged NAS drive via SMB v1, but even this failed. Whilst I could mount the share without error, and navigate the directory structure, anything that I tried to copy from the media card failed with "Input/Output Error".

Finally, I settled on connecting to the MG5350 via USB. On the MG5350 side, you don't need to do anything special at all. All of the MG5350 settings are set to default values, with the possible exception of the printer's Wifi being disabled, as I don't need to connect to it over Wifi at all, but you could just as easily leave that on. All you need to do is remember to select "Memory Card" as the destination when saving your scan (and have a memory card inserted in the card reader, of course).

## The RPi

The RPi is pretty much an out-of-the-box install of the latest Raspberry Pi OS Lite, with the SMB client and server software already up and running. As mentioned previously, I'd already setup this RPi to talk to a NAS over SMB, and share some other folders over SMB, so it's possible that if you're starting from scratch you'll need to do more than I've described here. Nonetheless, all of the configuration that's specific to the scanner-file-sharing should be here. If you need a guide to get you going with SMB, then I'm pretty sure that I followed https://pimylifeup.com/raspberry-pi-samba/ but Google will find you plenty of others too.

## USBmount

USBmount is a package that automatically mounts USB drives when they're inserted into the RPi. As mentioned above, you'll need to install usbmount using:

```
wget https://github.com/nicokaiser/usbmount/releases/download/0.0.24/usbmount_0.0.24_all.deb
dpkg -i usbmount_0.0.24_all.deb
```

(Note that this installs v0.0.24, you may want to visit the official github page and download a newer version if one is available by the time you're reading this)

Whilst it is possible to install USBmount using sudo apt install usbmount, the current version (at the time of writing) in the apt repositories is a couple of versions behind latest, and includes bugs that caused me issues.

You can find out more about USBmount on GitHub, but in a nutshell, USBmount will automatically mount any USB drive that is inserted into the RPi under /media/usb0 (unless it has already mounted something else there, in which case it'll be usb1 or usb2 or… up to, I think, usb7).

Other than installing the latest GitHub version of USBmount, and not the apt version, I haven't had to tweak or reconfigure USBmount in any other way.

Because I'm not plugging anything else into my RPi, I've just assumed that the drive always appears under /media/usb0. Depending on your setup, this may not work for you. We'll revisit this when we discuss the scanner-sync.sh script later, but for now it's perhaps useful for me to point out that USBmount also adds a simlink in /var/run/usbmount/ based on the name of the device, so you might find that a more consistent approach for any scripting that you need to do.

## The Samba share

Having successfully mounted the scanner's memory card, you might think that the easiest thing to do would be to share the mounted directory directly; i.e. setup an SMB share for /media/usb0. I did. It turns out that this is a terrible idea.

I had so many problems with this that I'm not sure I can remember them all. If the SMB share was running before you mounted the USB drive, then the SMB clients couldn't see the files. If the SMB share was started after the USB drive was mounted then the SMB clients could see the files, but if the USB drive re-mounted (e.g. if you turned the scanner off and on again) you had to restart the SMB share to update the list of files, at which point the SMB clients threw errors about "stale files". Finally, through all of this, I found that, if you mounted the memory card *and then scanned more files to the memory card* these new files *didn't* appear in the list of files in /media/usb0; you had to re-mount the memory card in order for the new files to be picked up (I suspect that this is actually a limitation of the scanner, and not USBmount).

In the end, I setup Samba to share a completely different directory. I happened to choose a directory that's backed by my NAS, but you could just as easily share a directory from the RPi itself, so long as you have enough free storage on your RPi memory card; or you could plug in a USB drive, let USBmount mount it for you, and share your files from there. The important point is that it needs to be a "proper" directory, not something provided by a scanner that's being turned off and on all the time, and doesn't have any weird behaviour around not showing you new files.

As described above, you can use this snippet in your /etc/samba/smb.conf to share a directory, /home/pi/max/scanner in this case, as an SMB share called scanner.

```
[scanner]
path = /home/pi/max/scanner
writeable=no
public=no
browseable=yes
```

The only slight problem is, of course, that you won't have any scanned documents in `/home/pi/max/scanner` yet…

## The script

This is the final piece of the puzzle; the `scanner-sync.sh` script is responsible for copying scanned files from the scanner's memory card to the directory that SMB is sharing to the internal network. No fancy symlinks, just good old fashioned file-copy operations.

Rather than duplicate the entire script here, which you can easily copy from above, I'll step through each of the sections and explain why I've written them that way.


---
```
#!/bin/bash
set -e
```
This is a bash script. If you'd like to learn more about bash, then I recommend Learn Bash the Hard Way. This script will stop as soon as something goes wrong.


---
```
if [[ $EUID -ne 0 ]]; then
   echo "This script must be run as root" 
   exit 1
fi
```
This script has to be run with root privileges; neither rsync nor udevadm (which we call later on) will work if we don't have root privileges.


---
```
source_dir=/media/usb0/CANON_SC/
destin_dir=/home/pi/max/scanner/
```
Here, we define our source directory (the scanner's memory card) and our destination directory (the mounted NAS drive). It's not particularly important, but you might notice that I don't actually copy *everything* off the memory card, just everything in the CANON_SC directory, as this is where the scanner stores its files.

This is where my first main hack comes in; I've *assumed* `usb0` in my script because I'm not mounting anything else. I should probably use the simlink in `/var/run/usbmount/` that's based on the name of the device, as that won't change (unless I plug in two MG5350s, which is pretty unlikely).


---

Next, we check that both the source and destination directories exists, as if either of them are missing, there's no point trying to copy files between them.
```
if [[ ! -d $source_dir ]]
then
 echo "$source_dir is not an existing directory"
 echo "Scanner is probably off"
 exit 1
fi
if [[ ! -d $destin_dir ]]
then
 echo "$destin_dir is not an existing directory"
 echo "NAS is probably off"
 exit 1
fi
```

---

Having completed all of our checks, I now do an initial copy from the media card to the NAS. I use rsync which, if you haven't heard of it, is pretty much built for this kind of thing.

```
echo "Initial sync of files from scanner to NAS"
rsync -a --no-g --no-o --update $source_dir $destin_dir
```

The -a tells rsync to "archive" the source directory, i.e. copy everything in it, including subdirectories. --no-g and --no-o stop rsync from trying to match the group and owner attributes on the destination drive (it's an SMB share so those things aren't supported).

Perhaps the most important attribute here is --update which tells rsync to only copy files that either don't exist in the destination directory, or have changed since they were copied. Without this, rsync will copy everything regardless of whether or not it's been copied before, and so will be much slower.

Please note, you must use --update and not --skip-existing. --skip-existing will literally skip any file that already exists in the destination directory, regardless of whether the file sizes are different. It's possible for the scanner to be part-way through saving a file at the same time as you're copying it, and if this happens, you'll end up with an empty or corrupt file in the destination directory. With --skip-existing, the next run of rsync won't copy over the non-empty/non-corrupt file as a file with that name already exists in the destination directory. With --update, rsync will notice that the file sizes are different and copy over the complete file.

So whilst with --update you still get in the situation where blank or corrupt files appear in your destination directory, these then get fixed by re-running the script once the scanner is finished saving the file. I'm not entirely happy about this behaviour, and think that I could do better, but I'll come back to that later.


---

As mentioned previously, if we just leave the media card mounted forever, then the RPi never sees any new files that are saved to the card. To solve this, we programmatically "unplug" and "plug back in" the USB drive (although obviously this doesn't physically remove the USB cable from the RPi, it just triggers the code that would run if we did).

```
echo "Reloading media card to pick up new files"
echo "Removing..."
udevadm trigger --action=remove /dev/sda1
sleep 10
echo "Re-adding..."
udevadm trigger --action=add /dev/sda1
sleep 10
```

---

Finally, we re-run the rsync operation to pick up any newly-saved files that we couldn't see before.

```
echo "Re-syncing files from scanner to NAS, to pick up new changes"
rsync -a --no-g --no-o --update $source_dir $destin_dir
```

To be honest, running rsync both before and after re-mounting the memory card is probably overkill, and I should only run it the once here. At the time, I was worried that the re-mounting operation would fail, and so I thought it best to copy the files that I had access to at the time, rather than ending up copying nothing at all. As it is, I think that I shall remove the first rsync and just rely on this one.


---

Finally finally, we notify the world that we're done, and cleanly exit the script
```
echo "Done"
exit 0
```

---

## Cron
Yes, I know I said that the script was the final piece of the puzzle, but I can't be sitting there running sudo /home/pi/scanner-sync.sh every time my kids scan a file.

Instead, I've configured cron to run the script every minute as root, by running `sudo crontab -e` and adding the following line at the end:

`* * * * * /home/pi/scanner-sync.sh 2>&1 | /usr/bin/logger -t scanner_sync`

If you don't know anything about cron, then the official RPi documentation may help. If you know a little bit about cron, then you may well wonder what this bit does:
`2>&1 | /usr/bin/logger -t scanner_sync`

Here, we are piping stdout and stderr to logger. Without this, cron will try to email us anything that the script outputs, so all those lovely `echo "..."` messages get rolled up into an email that then gets stuck in the RPi mail system somewhere (because no-ones actually configures their RPi for email, right?!). By piping our output in this way, all of those lovely `echo "..."` messages will appear in the system logs, specifically /var/log/messages and we can read them with `tail -f /var/log/messages`, where we'll see something like:

```
Jan 13 21:33:01 raspberrypi scanner_sync: Initial sync of files from scanner to NAS
Jan 13 21:33:01 raspberrypi scanner_sync: Reloading media card to pick up new files
Jan 13 21:33:01 raspberrypi scanner_sync: Removing...
Jan 13 21:33:02 raspberrypi usbmount[2621]: executing command: umount -l /media/usb0
Jan 13 21:33:02 raspberrypi usbmount[2621]: executing command: run-parts /etc/usbmount/umount.d
Jan 13 21:33:11 raspberrypi scanner_sync: Re-adding...
Jan 13 21:33:14 raspberrypi usbmount[2638]: executing command: mount -tvfat -osync,noexec,nodev,noatime,nodiratime /dev/sda1 /media/usb0
Jan 13 21:33:14 raspberrypi usbmount[2638]: executing command: run-parts /etc/usbmount/mount.d
Jan 13 21:33:22 raspberrypi scanner_sync: Re-syncing files from scanner to NAS, to pick up new changes
Jan 13 21:33:22 raspberrypi scanner_sync: Done
```

That's it! You should now be all set, with your script copying over all scanned files when the scanner is on, and just silently ticking over when the scanner is off.

# Limitations and further work
As Andrew Clay Shafer once said (albeit probably paraphrasing someone else) "broken gets fixed, crappy lives forever", so there's a good chance that what I have now will never get improved upon. Nonetheless, it would be nice to address the following "features".

## Re-enable power saving
The MG5350 is configured to turn itself off after 15 minutes of inactivity. Unfortunately, because the script is re-mounting the memory card every minute, this counts as activity, and the scanner never shuts down. I'm hoping that this will get resolved as a bi-product of resolving other issues.
## Smarter copying of files from the memory card
Rather than copy empty/corrupted files from the card, it would be better if the script detected these and waited before copying them across. There is a --dry-run option for rsync, and I wonder if I can use this to determine which files are due to be copied and, from there, whether or not they are ready to copy. For example, if there is more than one file to copy, we know that we are safe to copy the everything except the last file, because only the last file can be in this partially-written state. I need to do some more thinking and experimentation to find out what's possible here.
## Smarter enabling/disabling of the script
USBmount provides a hook for me to run a script every time a device gets mounted. Rather than driving the script from cron, I wonder if I could start the script when the memory card gets mounted (i.e. when someone turns the scanner on), and then stop the script once the scanner is off again.
The slight problem with stopping the script once the scanner is off again is that the scanner won't turn off unless I first turn off the script - I need to stop periodically re-mounting the memory card so that the scanner has 15 minutes of idle time to turn itself off. If I do that, and someone scans a file within that 15 minutes of idle time, then my script won't detect the new file and upload it to the shared directory. I need to think about what I can do to minimise the chances of that happening. One possible option is to detect when some other threshold is reached, such as the script has been running for x amount of time and no new files have been copied, or it is past 10pm when no-one should be doing any scanning anyway.
Of course, I could teach the "users" that if a scanned file hasn't appeared on the shared drive then they just need to turn the scanner off and on again, which may be OK, but I am a little reluctant to teach them this as "normal" behaviour, as they will no doubt end up just turning it off and on again after every scan.
## Stop using the NAS to host the copied files
Whilst the NAS does work, I am conscious that my continual rsync-ing to it means that it's kept out of its usual idle state, and drawing more power than it ought. For me, the NAS is more suited to hosting music and video files that I access sporadically, and so I will probably try attaching a USB drive to the RPi itself, and host the copied files from there.

# Sources

I found these links particularly helpful:
- Check if a script is running as root, and exit if not:
  - https://askubuntu.com/questions/15853/how-can-a-script-check-if-its-being-run-as-root
- Pipe stderr and stdout from a cron job to syslog, and then read the output via tail -f /var/log/messages:
  - https://serverfault.com/questions/137468/better-logging-for-cronjobs-send-cron-output-to-syslog
- Bash conditional expressions - which flag checks for existing directory?
  - https://www.gnu.org/software/bash/manual/html_node/Bash-Conditional-Expressions.html
- rsync only new files, using --ignore-existing /--update flag
  - https://unix.stackexchange.com/questions/67539/how-to-rsync-only-new-files
- Creating a systemd service (I didn't do this, but might someday)
  - https://www.raspberrypi.org/documentation/linux/usage/systemd.md
  - https://fedoramagazine.org/what-is-an-init-system/
- Get rid of "permission denied" errors when rsync'ing to a windows share (--no-o and--no-g):
  - https://serverfault.com/questions/364709/how-to-keep-rsync-from-chowning-transferred-files
- Usbmount documentation for the udevadm trigger command
  - https://github.com/rbrito/usbmount
- Setup ssh on RPi (not strictly needed as part of this tutorial but I had to move my pi to be next to the scanner and so had to switch from keyboard/mouse connectivity to ssh):
  - https://www.raspberrypi.org/documentation/remote-access/ssh/unix.md
- Canon: Using the card slot over the network:
  - https://support.usa.canon.com/kb/index?page=content&id=ART114054
  - But then this gave input/output errors when trying to copy the files from the machine, so I just include for historical interest only.