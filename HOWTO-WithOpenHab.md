# RasLIRC HowTo for OpenHAB

## Table of Contents
  * Introduction & Background
  * My Hardware
  * OpenHAB Config
  * Control Interface
 
---
## Introduction & Background
I wanted a cost effective way to control my home office setup, without buying expensive custom hardware and software.  My setup consists of 3 TVs around the room, plus several input devices controlled through an HDMI matrix switch.  There are big name companies that specialize in this type of thing, but I wanted one I could do on my budget and as a project.  So my needs were simple:
  * Keep the project on a reasonable low budget
  * Use FOSS software wherever possible
  * Contribute back the community with my setup so others can do similar things w/o my learning pains
 ---
## My Hardware
As you've (hopefully) seen in the Readme.md, the IR control hardware was home brewed by me, based on some learnings from the Internet.  For the full solution though, I have:
  * Several of the "RasLIRC" devices I built in this project, all powered via PoE splitters running from my PoE switch
  * A Raspberry Pi 3 Model B running OpenHabian (OpenHab for Raspberry Pi)
  * A Kindle Fire HD 10 (with Google Play side-loaded running the Fully Kiosk app / browser)
 
I'm controlling:
  * 3 Samsung TVs
  * 1 LG Soundbar
  * 1 AV Access HDMI Matrix Switch

## OpenHAB Config
This was fairly straight forward once I got the LIRC boxes working properly.  Before you do any of this, make sure your LIRC binding is enabled.

#### Things configuration
First, in your Things configuration you need to define the binding.  This registers the LIRC "bridge" and each remote it has under it:

```
Bridge lirc:bridge:lirc_officetv1 [ host="192.168.0.211", portNumber="8765" ] {
    Thing remote SamsungTV [ remote="Samsung_BN59-01224C" ]
}
```

  * `lirc:bridge:lirc_officetv1` - "lirc_officetv1" is the name I'm assigning to the bridge device
  * `host="192.168.0.211", portNumber="8765"` - Is the IP and port the LIRC service is listening on on the remote Raspberry Pi
  * `Thing remote SamsungTV` - "SamsungTV" is the name I'm assigning to this particular remote database...
  * `remote="Samsung_BN59-01224C"` - Defines the remote database to use for the above named remote

#### Items configuration
Next, you need to define the Items for the remote controls.  I did this with two items, so I can inject extra steps and logging.  In your Items file:

```
String Office_TV1_Remote_Control				"TV1 Remote Control Virtual Item"
String LIRC_Office_TV1_Samsung { channel="lirc:remote:lirc_officetv1:SamsungTV:transmit" }
```

  * `Office_TV1_Remote_Control` is a virtual item that I will use for all of the incoming commands (more on this in the Rules configuration)
  * `String LIRC_Office_TV1_Samsung` is the item that gets bound to the `lirc_officetv1` bridge, the `SamsungTV` remote, configured to `transmit`

#### Rules configuration
Lastly, you need to setup your Rules to control the device.  At least, this is how I did it.  You can have your OpenHAB interface send commands directly to the LIRC Item you configured, but I wanted the ability to string commands and inject logging.  That's the point of the virtual item.  So in my Rules file (abbreviated for example purposes):

```
rule "Office TV1 Remote Control"
when
  Item Office_TV1_Remote_Control received command
then
    switch (receivedCommand){
        case "KEY_POWER": {
        	logInfo("RULE", "Office_TV1_Remote_Control - KEY_POWER")
			sendCommand(LIRC_Office_TV1_Samsung, "KEY_POWER")
        }
    }
end
```

So when the Office_TV1_Remote_Control virtual item gets a command from my interface, it looks at the command and does different things based on it.  In this case, if the command is "KEY_POWER" it will log that it is sending the command, then send the `KEY_POWER` command to the `LIRC_Office_TV1_Samsung` Item.

## Control Interface
My control for all of this is a Kindle Fire HD 10 with the Fully Kiosk browser.  It displays the HabPanel UI that comes with OpenHAB.  To hold it up and charge it, I use the new Show Mode Charging Dock.

IMAGE HERE


