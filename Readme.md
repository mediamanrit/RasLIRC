# RasLIRC
## Table of Contents
  * Introduction & Background
  * Hardware
  * Initial OS Config
  * LIRC Config
---
## Introduction & Background

RasLIRC came from my need to have OpenHAB control multiple devices in my home office, which can only be controlled through IR.  While OpenHAB can have LIRC running on it directly, I instead opted to build multiple remote devices to do the IR work for OpenHAB.

As I attempted to learn how to do this, there were lots of different resources, many saying things that were contradictory to each other.  Further complicating matters, they all were for old versions of LIRC, and the newer versions (I'm using 0.9.4c) require slightly different configuration tasks.

This process will explain:
  * How to build the Raspberry Pi Hardware
  * How to do initial configuration of the OS
  * How to install/configure LIRC
  * (Bonus) How to connect the devices to OpenHAB
 
---
## Hardware

### Materials

#### Raspberry Pi
I based my project on several Raspberry Pi 3 Model B and Raspberry Pi 3 Model B+ devices.  You can get these from many different places, though I prefer my local MicroCenter.  For mail order / Internet shopping I prefer [Adafruit](www.adafruit.com) .

#### Storage
Using the right MicroSD card on a Raspberry Pi is critical.  If you don't use a good card it will effect the stability of the device.  I use [Samsung 32GB MicroSD](https://www.amazon.com/Samsung-MicroSD-Adapter-MB-ME32GA-AM/dp/B06XWN9Q99/ref=sr_1_2) cards almost exclusively.

#### Case
This was a struggle, finding a case that was large enough to house the desired 1/8" jacks (more on that later).  The modular cases by ModMyPi [here](https://www.modmypi.com/raspberry-pi/cases-183/raspberry-pi-b-plus2-and-3-cases-1122/plastic-cases-1142/modmypi-modular-rpi-b-plus-case-black) worked great.  I used the base case with one extender.  I purchased these parts (the case, extender ring, screws, and port covers) from [PiShop.us](https://www.pishop.us/product/modmypi-modular-rpi-b23-case-black/).

#### Power Supply
You can use any old USB power supply that's rated for 2.5A, [like this one](https://www.adafruit.com/product/1995).  For my purposes, I wanted something that was less intrusive at the location where the Pi will go, so I went with a [POE injector](https://www.amazon.com/gp/product/B07CNKX14C/ref=oh_aui_search_detailpage?ie=UTF8&psc=1).  The Raspberry Pi 3 Model B+ supports a POE hat/module, but that makes it more involved to use the circuit board components.

#### IR Electronics
This is the longest list.  This is the list of electronics I used to make the IR functionality
  * [Adafruit Perma-Proto Hat](https://www.adafruit.com/product/2310)
  * [IR Receiver Sensor - TSOP38238](https://www.adafruit.com/product/157)
  * [NPN Transistors](https://www.adafruit.com/product/756)
  * [10k Ohm Resistor](https://www.adafruit.com/product/2784)
  * [1/8" Panel Mount Jack](https://www.adafruit.com/product/3692)
  * [Stranded Wire](https://www.adafruit.com/product/3111)  
  * [Solid Wire](https://www.adafruit.com/product/1311) (Note that you could do the whole project with stranded wire, but I use both)
  * [IR Emitter](https://www.amazon.com/Cmple-Extender-Stick-Infrared-Extension/dp/B004WLA25G/ref=sr_1_5)
  
 ### Assembly

1. Start with the basics.  Put the Raspberry Pi in the case.
2. Assemble the circuit board / Perma-Proto Hat.  This took me 10-20 minutes.  For starters, here's a closeup image of the board:

![Inside photo](https://github.com/mediamanrit/RasLIRC/raw/master/Resources/Inside%20Case.jpg)

![Board Close-up](https://github.com/mediamanrit/RasLIRC/raw/master/Resources/Board%20Close-Up.jpg)

And here is the circuit diagram

![Schematic](https://github.com/mediamanrit/RasLIRC/raw/master/Resources/RasLIRC%20Schematic.png)

3. Place the Hat on the Raspberry Pi, making sure to make sure all 40 pins make contact.
4. Drill holes in the cover for the 1/8" jacks, as well as a hole directly over where the IR receiver is (for when you want to learn IR codes).

![Case Photo](https://github.com/mediamanrit/RasLIRC/raw/master/Resources/Case.jpg)

5. Place the cover on the case.
6. Admire your handy work!

## Initial OS Config
This section will cover what you need to do to load the OS onto the Pi, then do some very basic configurations to get it going.

### Install the OS
Raspbian is the go-to standard for the OS on a Raspberry Pi.  [Go here](https://www.raspberrypi.org/downloads/raspbian/) to download it.  For this use, I strongly recommend the "Lite" version, as it's designed for simple projects (read: doesn't come with a lot of junk).  On that page there is also a link to the Installation Guide.  Read that guide for instructions on how to write the image to your MicroSD card.

### Power it up!
Once the MicroSD card has the image written to it, install it onto the Raspberry Pi.  Next, connect up your Ethernet, monitor, and USB Keyboard.  Once those are connected, you can plug in the power supply.

The Pi will go through a fast initial boot, then immediately reboot to resize the image to take up the entire MicroSD card.  This is normal!  Don't touch anything until you see a login prompt.

### Initial OS Configuration
After the first boot of the Raspberry Pi, we're ready to get going:
1. Login with the user "pi" and the default password of "raspberry"
2. Type `passwd` and press enter.  This will walk you through resetting the password on the "pi" account.
3. Type `sudo apt-get install ansible`.  This will download the Ansible tool, which we will use to configure the rest of the device.  You will get prompted for your password, then again to confirm the installation.  It will install a lot more then Ansible since Ansible itself has many dependencies.

## LIRC Config
1. Download the Ansible playbook I made (in this repository) for installing and configuring everything else.  Type `wget this`.
2. Lastly, run the playbook by typing `ansible-playbook RasLIRC-Wired.yaml`.  This will perform many steps, and list out each one as it goes.  When it's complete, it should reboot the pi.  If not, type `sudo reboot`.

LIRC is now installed, and in theory running!  Now, what do you do with it?  Remember, this guide is intended to have LIRC control a device.  That means you now need to give it IR codes to use for those devices.  The simplest method is to look in one of the remote control databases like [this one](https://sourceforge.net/p/lirc-remotes/code/ci/master/tree/remotes/) .  Remember that often times manufacturers use IR codes over and over again.  So while your particular remote control model may not be there, a similar one from the same manufacturer may work perfectly fine.

Once you find a file (or more) to try, you need to get them onto the pi.  If you are logged into the console of the pi already, the simplest way to do this is directly on the pi:
1. Put yourself into the configuration directory where the IR codes are stored: `cd /etc/lirc/lircd.conf.d`
2. Download a file from the repository.  For example: `sudo wget https://sourceforge.net/p/lirc-remotes/code/ci/master/tree/remotes/samsung/BN59-01224C.lircd.conf`
3. If the file you downloaded does not end in ".conf", it will be ignored.  Therefore, you have to rename it for LIRC to pick it up.  For example: `sudo mv someremote.lircd someremote.lircd.conf`
4. Once you get all the configuration files in place, restart LIRC: `sudo systemctl restart lirc`

Now we can finally test it out.  Before you do, some quick checks:
  * Make sure the device you're trying to control actually works with the original remote
  * Make sure the IR blaster is plugged into the 1/8" jack on the device you just built
  * Make sure the IR blaster is in the right place.  This is super critical.  There are several ways to accomplish this.  I generally start with a really bright flashlight, and move around the frame of the device to look for where the IR receiver is.  Once I think I've found it, I'll take the factory remote and put it right up to that spot (literally touching it) and try to control the device.  If it doesn't work, then keep moving to find the right spot.

We'll do some progressive tests now, working our way up to the actual control of the device:
1. `irsend list "" ""` - This will list out the remote controls that LIRC has registered.  These are the config files that you downloaded from the Internet above.  If one is missing, check to make sure it's named correctly and that you've restarted LIRC after it was put in place.
2. `irsend list Samsung_BN59-01224C "" ` - Replace `Samsung_BN59-01224C` with one of the remotes that came up in the list in step 1.  This will list out all of the codes for that remote control.  The left column will be the code in hex format, the button will be the right column.
3. `irsend list Samsung_BN59-01224C KEY_POWER` - This test is a bit redundant, but it doesn't hurt.  It ensures that LIRC can pull out the command for only one button.  In this case, `KEY_POWER`.
4. `irsend send_once Samsung_BN59-01224C KEY_POWER` - This will now send the KEY_POWER command once, from the Samsung_BN59-01224C database.  If all goes well, the device should respond.

At this point LIRC is now working.  How you interface with it is up to you!

But wait.  There's more!  In [HOWTO-WithOpenHab.md](https://github.com/mediamanrit/RasLIRC/blob/master/HOWTO-WithOpenHab.md) I'll explain how I interfaced this with OpenHab.

There's also a troubleshooting guide, [Troubleshooting.md](https://github.com/mediamanrit/RasLIRC/blob/master/Troubleshooting.md), where I'll keep my notes and troubleshooting steps.







