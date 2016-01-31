---
layout: post
published: true
title: Project pfSense - x750e Further Enhancements

tags:
- pfSense
- firebox
---

This post relates directly to the (somewhat outdated) [X-Core-e > Further Enhancements](https://doc.pfsense.org/index.php/PfSense_on_Watchguard_Firebox#Further_Enhancements_3) part of the pfSense Watchguard Firebox Wiki.

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

**TODO** Add photo

After replacing the original DIMM with one of my 2GB ones I booted my firebox. Once booted I launched the web interface to check my 2G.... 900MB of RAM?!?

**TODO** Add screenshot

After some more reading I figured out the board _does_ support up to 2GB but is limited to 1GB per DIMM. So I added my second 2GB DIMM, booted the system and sure enough.. 2007MB of memory was recognized!
NOTE: The DIMMs only support up to 1GB / DIMM so this setup is far from ideal!

**TODO** Add photo + Screenshot

## CPU

**TODO** write-up once received

### Installing the hardware

Today I received my new CPU so let's get cracking!

As you can read on the Wiki the board supports both the 130nm "Banias" and the 90nm "Dothan" familly. So I went ahead and ordered a "Dothan" Pentium M755 (SL7EM, 2GHz, 2MB L2 Cache) of of eBay for €10. Although it has the same FSB speed it supports Intel EnhanchedSpeedstep (more on this later), has 4x more L2 Cache and will increase the raw GHz with 53%!

With the case open I removed the over engineered heatsink. Underneath the heatsink we find the default Intel Celeron M320 (SL6N7, 1.3GHz, 512K L2 Cache). This CPU is part of the 130nm "Banias" family. 

**TODO** Add picture of old CPU

Replacing the CPU itself is straight forward: you unlock the old one, take it out, put in the new one, lock it, clean heatsink, apply cooling paste, re-mount the heatsink, Done! 

**TODO** Add picture of new CPU

Once the new CPU is in place you need to adjust both DIP switches on the motherboard to adjust the voltage and the FSB settings to match your new chip.

**TODO** Add pitures of DIP switches 

With all of that out of the way, time to put the new CPU to work! Remember, the firebox will beep on successfull POST. So if you boot it and it beeps.. great success! CPU recognized!

### Make it green(er)

Next step is to take advantage of the Intel Enhanched SpeedStep in order to reduce the power consumption.
The process to do so is described on the pfSense Wiki as well as on the [forum](https://forum.pfsense.org/index.php?topic=20095.msg161139#msg161139) but since your here I'll explain it :)

First we configure the timecounter to use the i8254 device. 
- In order to do so 'on-the-fly' you can run the following command in a shell prompt on your firebox:
```
sysctl kern.timecounter.hardware=i8254
```

I did add the setting to the system tunables:
- Navigate to System > Advanced > System Tunables
- Add the config from the command above
- **TODO** Add screenshot of settings to add
- Save and Apply config

Next we enable PowerD
- Navigate to System > Advanced > Miscellaneous
- Enable PowerD
- Save config

All that left is to force our firebox to use EST instead of ACPI or P4TCC. Only EST provides measurable power savings.
- Connect to your firebox: 
	- `ssh admin@<IP_Of_pfSense_Box>`

- Remount the filesystem as read-write
```
/etc/rc.conf_mount_rw
```

- Edit the /boot/loader.conf.local file
	- `vi /boot/loader.conf.local`

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
**TODO** Add screenshot of speedstep


## PSU

**TODO** investigate PicoPSU, order, document

## NIC LEDs

As written on the Wiki:

>The LEDs on the two sets of Marvell NICs do not behave as you might expect them to. In the default condition after install they will show only activity on the NIC rather than showing the LINK state as most other devices would. This is because the Marvell 88e8053 and 88e8001 both have programmable LED outputs and the default state is incorrect for the LED configuration in the Firebox. For a _much_ more complete explanation see the forum thread from [this](http://forum.pfsense.org/index.php/topic,20095.msg272392.html#msg272392) post onwards.

**TODO** Add gif of the problem

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
**TODO** Perform command on firebox and update output below
```
42ddd204adc647e5c35c4107b52d11bf  if_msk.ko
f7251a7d42cedb2f19723e39b9617935  if_sk.ko
```

- Correct the permissions on these files
```
chmod 555 if_*
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
**TODO** Add print of syslog output
```
Syslogoutput
```

## HDD Caddy & Extreme stuff

I haven't had a go (nor the desire to do so at the moment) on either one of these so I guess if you want more info on these you'll have to source the forums.

I hope this was usefull to anyone! Have fun with your enhanced Firebox.
