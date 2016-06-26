---
layout: post
published: true
title: Project pfSense - x750e Further Enhancements

tags:
- pfSense
- firebox
---

This post relates directly to the (somewhat outdated) [X-Core-e > Further Enhancements](https://doc.pfsense.org/index.php/PfSense_on_Watchguard_Firebox#Further_Enhancements_3) part of the pfSense Watchguard Firebox Wiki.

## Action log

I'll be updating this post as I go, so stay tuned!

```
10/01/2016: Initial creation of the document with draft layout
31/01/2016: Received new CPU, documented installation
26/06/2016: Ordered PicoPSU, added some small updates and pictures
```

## LCD enhancements

**TODO**

Write LCD-DEV package + cmdshell

## WGXepc (Arm/Disarm LED + Fan Speed)

**TODO**

Explain installation of WGXepc + config using cmdshell

Link to files (+copy files on git)

## RAM

The board does support up to 2GB DDR2 RAM as explained on the Wiki. Personally I had some 2GB DDR2-533 Kingston Value DIMMs laying around from an old desktop. For those interested in the details: KVR533D2N4K2/2G. 

So I opened the box and poped out the original DIMM which was an 512MB DDR2-533 Transcend DIMM.

![RAM_Modules]({{ site.baseurl }}/images/20160110/RAM_Modules.png "Top: Original 512MB DIMM | Bottom: New 2GB DIMM")

After replacing the original DIMM with one of my 2GB ones I booted my firebox. Once booted I launched the web interface to check my 2G.... 900MB of RAM?!?

![RAM_Upgrade_NOK]({{ site.baseurl }}/images/20160110/RAM_Upgrade_NOK.png "Only half of the DIMM is recognised")

After some more reading I figured out the board _does_ support up to 2GB but is limited to 1GB per DIMM. So I added my second 2GB DIMM, booted the system and sure enough.. 2007MB of memory was recognized!
_NOTE_: The DIMMs only support up to 1GB / DIMM so this setup is far from ideal!

![RAM_Upgrade_OK]({{ site.baseurl }}/images/20160110/RAM_Upgrade_OK.png "With 2x2GB DIMMs 2GB is usable")

## CPU

### Installing the hardware

Today I received my new CPU so let's get cracking!

As you can read on the Wiki the board supports both the 130nm "Banias" and the 90nm "Dothan" familly. So I went ahead and ordered a "Dothan" Pentium M755 (SL7EM, 2GHz, 2MB L2 Cache) of of eBay for €10. Although it has the same FSB speed it supports Intel EnhanchedSpeedstep (more on this later), has 4x more L2 Cache and will increase the raw GHz with 53.846153846%!

With the case open I removed the over engineered heatsink. Underneath the heatsink we find the default Intel Celeron M320 (SL6N7, 1.3GHz, 512K L2 Cache). This CPU is part of the 130nm "Banias" family. 

![Stock_CPU]({{ site.baseurl }}/images/20160110/SL6N7.png "The default SL6N7 CPU")

Replacing the CPU itself is straight forward: you unlock the old one, take it out, put in the new one, lock it, clean heatsink, apply cooling paste, re-mount the heatsink, Done! 

![New_CPU]({{ site.baseurl }}/images/20160110/SL7EM.png "New CPU, SL7EM")

Once the new CPU is in place you need to adjust both DIP switches on the motherboard to adjust the voltage and the FSB settings to match your new chip.

![DIP_Switch_1]({{ site.baseurl }}/images/20160110/DIP_Switch_1.png "First pair of DIP switches")
![DIP_Switch_2]({{ site.baseurl }}/images/20160110/DIP_Switch_2.png "Second pair of DIP switches")


With all of that out of the way, time to put the new CPU to work! Remember, the firebox will beep on successfull POST. So if you boot it and it beeps.. great success! CPU recognized!

### Making it green(er)

Next step is to take advantage of the Intel Enhanched SpeedStep in order to reduce the power consumption.
The process to do so is described on the pfSense Wiki as well as on the [forum](https://forum.pfsense.org/index.php?topic=20095.msg161139#msg161139) but since your here I'll explain it :)

First we configure the timecounter to use the i8254 device. 

- In order to do so 'on-the-fly' you can run the following command in a shell prompt on your firebox

```
sysctl kern.timecounter.hardware=i8254
```

- Then add these settings to the system tunables

	- Navigate to System > Advanced > System Tunables
	- Scroll all the way to the bottom and select the Add icon
	- Add the config from the command above
	- ![Kern_Timecounter]({{ site.baseurl }}/images/20160110/kern_timecounter.png "Configure the timecounter to use i8254")
	- Save and Apply config

Next we enable PowerD

- Navigate to System > Advanced > Miscellaneous
- Enable PowerD
- Save config

All that left is to force our firebox to use EST instead of ACPI or P4TCC. Only EST provides measurable power savings.

- Connect to your firebox: 

``` 
ssh admin@<IP_Of_pfSense_Box> 
```

- Remount the filesystem as read-write:

```
/etc/rc.conf_mount_rw
```

- Edit the /boot/loader.conf.local file:

```
vi /boot/loader.conf.local
```

- Add following lines at the bottom:

```
hint.p4tcc.0.disabled=1
hint.acpi_throttle.0.disabled=1
est_load="YES"
```

- Remount the filesystem as read-only

```
/etc/rc.conf_mount_ro
```

Now reboot your system and see SpeedStep in action on the dashboard!

![Speedstep]({{ site.baseurl }}/images/20160110/speedstep.gif "Animated gif of speedstep in action on the status dashboard")


## PSU

At the moment the main source of heat in the firebox is the powersuply. More heat means more cooling, more cooling means more energy usage which in turn means more power conumption.
So it's time the swap out the old power supply for something smaller, more performant and way cooler (in both ways :p).

I ordered a PicoPSU 120W and a powerbrick of 80W. I picked this up from the Nethelands from [Afuture.nl](https://www.afuture.nl/productview.php?productID=2438419) which had the best price at the time of writing. Those of you living in the US probably want to check out [mini-box.com](http://www.mini-box.com/s.nl/it.A/id.417/.f).

**TODO** PicoPSU:

- Order 
	- _PicoPSU 120W has been ordered._ Waiting for delivery to test it out.
- Install
	- Check power consumption difference
- Print 3D plate to close cover (Credits go to _PantsManUK_ on the pfSense Forums):
- ![Power_Connector_Plate]({{ site.baseurl }}/images/20160110/Power_Connector_Plate.png "3D design power connector plate")
- Document

## NIC LEDs

As written on the Wiki:

>The LEDs on the two sets of Marvell NICs do not behave as you might expect them to. In the default condition after install they will show only activity on the NIC rather than showing the LINK state as most other devices would. This is because the Marvell 88e8053 and 88e8001 both have programmable LED outputs and the default state is incorrect for the LED configuration in the Firebox. For a _much_ more complete explanation see the forum thread from [this](http://forum.pfsense.org/index.php/topic,20095.msg272392.html#msg272392) post onwards.

![Broken_LED_driver]({{ site.baseurl }}/images/20160110/LED_Broken.gif "LEDs only show link status") 

The steps to update the drivers on the wiki are still valid though the links to the files are outdated. The newer versions of pfSense are based on FreeBSD 10. The files on the Wiki are for the older versions of pfSense _(2.1.*)_ which were based on BSD 8.3. Lucky for us _ceama_ did update the kernel modules and recompiled them for BSD version 10. He made these available on the forum via [this post](https://forum.pfsense.org/index.php?topic=20095.msg460754#msg460754). As a reference I will copy these files on my git repository as well, but credits go to _ceama_ over at the pfSense community.

So in order to fix the NIC LEDs behaviour:

- Connect to you Firebox using ssh

```
ssh admin@<IP_Of_pfSense_Box> 
```

- Remount the filesystem as read-write

```
/etc/rc.conf_mount_rw
```

- Fetch the kernel modules to **/boot/modules/**

```
fetch -o /boot/modules/if_sk.ko  http://arganox.github.io/files/firebox/NIC/if_sk.ko

# Not needed for X550e (as they do not have the msk interfaces)
fetch -o /boot/modules/if_msk.ko http://arganox.github.io/files/firebox/NIC/if_msk.ko
```

- Check the MD5 of the files

```
md5 if_*.ko

# Output should match:
MD5 (if_msk.ko) = 42ddd204adc647e5c35c4107b52d11bf
MD5 (if_sk.ko) = f7251a7d42cedb2f19723e39b9617935

```

- Correct the permissions on these files

```
chmod 555 if_*.ko
```

- To make the kernel modules load on boot add following lines to **/boot/loader.conf.local**
	- **NOTE:** If you're using windows use [Notepad++](http://notepad-plus-plus.org/) cause notepad's CR/LF might brake things!

```
if_sk_load="yes"
if_msk_load="yes"
```

- Remount the filesystem as read-only

```
/etc/rc.conf_mount_ro
```

- Reboot the firebox

```
reboot
```

- To check if the modules are loaded correctly: Look at the LEDs... duh! or if you're to lazy to turn your head (Can't blame you! :)) you can check the syslog:

```
dmesg|grep LED

# Output should show:
mskc0: <Marvell Yukon 88E8053 Gigabit Ethernet (LED mod 2.2)> port 0x8000-0x80ff mem 0xd0020000-0xd0023fff irq 16 at device 0.0 on pci1
mskc1: <Marvell Yukon 88E8053 Gigabit Ethernet (LED mod 2.2)> port 0x9000-0x90ff mem 0xd0120000-0xd0123fff irq 17 at device 0.0 on pci2
mskc2: <Marvell Yukon 88E8053 Gigabit Ethernet (LED mod 2.2)> port 0xa000-0xa0ff mem 0xd0220000-0xd0223fff irq 18 at device 0.0 on pci3
mskc3: <Marvell Yukon 88E8053 Gigabit Ethernet (LED mod 2.2)> port 0xb000-0xb0ff mem 0xd0320000-0xd0323fff irq 19 at device 0.0 on pci4
skc0: <Marvell Gigabit Ethernet (LED mod 2.2)> port 0xc000-0xc0ff mem 0xd042c000-0xd042ffff irq 16 at device 0.0 on pci5
skc1: <Marvell Gigabit Ethernet (LED mod 2.2)> port 0xc400-0xc4ff mem 0xd0420000-0xd0423fff irq 17 at device 1.0 on pci5
skc2: <Marvell Gigabit Ethernet (LED mod 2.2)> port 0xc800-0xc8ff mem 0xd0424000-0xd0427fff irq 18 at device 2.0 on pci5
skc3: <Marvell Gigabit Ethernet (LED mod 2.2)> port 0xcc00-0xccff mem 0xd0428000-0xd042bfff irq 19 at device 3.0 on pci5
```

## HDD Caddy & Extreme stuff

I haven't had a go (nor the desire to do so at the moment) on either one of these so I guess if you want more info on these you'll have to source the forums.

I hope this was usefull to anyone! Have fun with your enhanced Firebox.
