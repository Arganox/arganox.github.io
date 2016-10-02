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
26/06/2016: Ordered PicoPSU
            Added some small updates and pictures
11/09/2016: Ordered IDE 44pin to SATA connector pieces
            Updated WGXepc installation instructions
13/09/2016: Installed PicoPSU, documented installation
            Added LCD-dev package instructions
02/10/2016: Installed SATA HDD drive
            Documented installation
```
----

## LCD enhancements

**NOTE** As of version 2.3 of pfSense LCDproc-DEV package seems to be removed but should be [installable manually](https://forum.pfsense.org/index.php?topic=44034.msg645684#msg645684).

Aaa pfSense running on your watchguard, looking nice!  
Flashing lights and an LCD which displays... absolutely nothing usefull. **sigh...**

Out of the box the LCD of the watchguard will be displaying **PFSense _BIOS-VERSION_ Booting OS...**  
We can use the LCD to display some live stats regarding our watchguard. This can be accomplished with the LCDproc-dev package which is [well described](https://forum.pfsense.org/index.php?topic=7920.msg344513#msg344513) in the forums by _stephenw10_.  
Lets go!

- First navigate to System -> Packages. From there click on the “Available Packages” tab. Then search and install “LCDproc-dev” and “Shellcmd“ (I'll be using this in the next part as well). Make sure you select the **dev** version of LCDproc as it includes working drivers for the watchguard LCD. Shellcmd will allow us to execute commands on boot. So here we'll be using it to start the LCDproc service once installed and configured.

- Once these packages are installed, navigate to Services -> LCDproc. Select/change the following: ‘Enable LCDproc at startup’ yes | Com port – Parallel Port 1 | Display Size – 2×20 | Driver – Watchguard Firebox with SDEC.

- By doing this the package will generate the lcdd.conf file which we will be copying to /conf. By doing this via the web GUI there is no need to remount the filesystem in RW. Go to Diagnostics: Command Prompt and run:

```
cp /usr/local/etc/LCDd.conf /conf
``` 

- Now go back to Services -> LCDproc, uncheck 'Enable LCDproc at startup' and set Com Port to 'none'. You must set the com port as none, that's what the LCDproc-dev config script looks for before it removes the RC start-stop scripts.

- To start the LCDproc server and client with every boot navigate to Services -> shellcmd and add the following commands:

```
#Start the LCDproc driver with the confi we created
/usr/bin/nice -20 /usr/local/sbin/LCDd -r 0 -c /conf/LCDd.conf > /dev/null &

#Select the screens you would like to display
/usr/bin/nice -20 /usr/local/bin/lcdproc C T U & 
```

- Reboot and look at the LCD to test the config

----

## WGXepc (Arm/Disarm LED + Fan Speed)
### Installing the base script

The WGXepc script can be used to control the fan speed, Arm/Disarm LED, LCD backlight. It can also read the CPU temperature.
So lets get installing!

- Remount the filesystem as read-write:

```
/etc/rc.conf_mount_rw
```

- Download the script to the /conf folder:

```
fetch -o /conf http://arganox.github.io/files/firebox/WGXepc/WGXepc
```

- Set permission so the script can be executed:

```
chmod 0755 /conf/WGXepc
```

- Test the program

```
[2.3.2-RELEASE][admin@pfSense.localdomain]/conf: ./WGXepc 
Found Firebox X-E
WGXepc Version 1.1 27/11/2014 stephenw10
WGXepc can accept two arguments:
 -f (fan) will return the current and minimum fan speed or if followed
    by a number in hex, 00-FF, will set it.
 -l (led) will set the arm/disarm led state to the second argument:
    red, green, red_flash, green_flash, red_flash_fast, green_flash_fast, off
 -b (backlight) will set the lcd backlight to the second argument:
    on or off. Do not use with LCD driver.
 -t (temperature) shows the current CPU temperature reported by the
    SuperIO chip. X-e box only.
Not all functions are supported by all models
```

- Remount the filesystem as read-only:

```
/etc/rc.conf_mount_ro
```

### Automatic execution on boot

In order to have these WGXepc commands executed on boot you can use the shellcmd package.

- If you haven't already done so in the first part of this post, install the shellcmd package.
This can be done by navigating to System > Packages from there click on the "Available Packages" tab.

![PackageManager]({{ site.baseurl }}/images/20160110/Packages.png "Package manager in the Web GUI.")

- Next search for the shellcmd package and install it

- Once installed you'll be able to find the new package under Services > Shellcmd. Navigate here in order to configure the WGXepc command to execute on boot.

![Shellcmd]({{ site.baseurl }}/images/20160110/shellcmd_menu.png "Shellcmd service in the Web GUI.")

- Add all the command you want to be executed on boot. eg: On boot, set the fan speed to 10:

![Shellcmd_fanspeed]({{ site.baseurl }}/images/20160110/shellcmd_WGXepc.png "Shellcmd to set the fan speed to 10 on boot.")

----

## RAM

The board does support up to 2GB DDR2 RAM as explained on the Wiki. Personally I had some 2GB DDR2-533 Kingston Value DIMMs laying around from an old desktop. For those interested in the details: KVR533D2N4K2/2G. 

So I opened the box and poped out the original DIMM which was an 512MB DDR2-533 Transcend DIMM.

![RAM_Modules]({{ site.baseurl }}/images/20160110/RAM_Modules.png "Top: Original 512MB DIMM | Bottom: New 2GB DIMM")

After replacing the original DIMM with one of my 2GB ones I booted my firebox. Once booted I launched the web interface to check my 2G.... 900MB of RAM?!?

![RAM_Upgrade_NOK]({{ site.baseurl }}/images/20160110/RAM_Upgrade_NOK.png "Only half of the DIMM is recognised")

After some more reading I figured out the board _does_ support up to 2GB but is limited to 1GB per DIMM. So I added my second 2GB DIMM, booted the system and sure enough.. 2007MB of memory was recognized!
_NOTE_: The DIMMs only support up to 1GB / DIMM so this setup is far from ideal!

![RAM_Upgrade_OK]({{ site.baseurl }}/images/20160110/RAM_Upgrade_OK.png "With 2x2GB DIMMs 2GB is usable")

----

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

----

## PSU

At the moment the main source of heat in the firebox is the powersupply. More heat means more cooling, more cooling means more energy usage which in turn means more power consumption.
So it's time to swap out the old power supply for something smaller, more performant and way cooler (both literally and nerdly :)).

I ordered a PicoPSU 120W and powerbrick kit. I picked this up in the Nethelands from [Afuture.nl](https://www.afuture.nl/productview.php?productID=2438419) which had the best price at the time of writing. Those of you living in the US probably want to check out [mini-box.com](http://www.mini-box.com/s.nl/it.A/id.417/.f). I bought an extra [P4 Male to Molex](https://www.afuture.nl/product/137001/delock-cable-p4-male-%3E-molex-4pin-male) connector since the PicoPSU didn't came with a P4 connector for the motherboard. I also bought one [P4 Male to P4 MAle](https://www.afuture.nl/product/151887/delock-power-cable-p4-malefemale) connector so I could re-use the power switch.

After a quick test of the PicoPSU it was time to install this little PSU. This step requires no specific info just open the firebox, disconnect the old bulky PSU, unscrew it, take it out and toss it away! Next you connect the PicoPSU and you're all set. You can now boot your firebox using your new PicoPSU which will lower it's power usage to somwhere around 25-30W. Overal I messured a saving of approximately 10W both during boot and operation.

I figured while I'm at it I might as well make this slik looking both inside and out. Big thanks to _PantsManUK_ on the forums. He made a 3D design for a cover plate that fits the firebox perfectly! So I printed [his design]({{ site.baseurl }}/files/firebox/PowerConnectorPlate/firebox-power-connector-blanking-plate.zip) which I uploaded here for reference.
![Power_Connector_Plate]({{ site.baseurl }}/images/20160110/Power_Connector_Plate.png "Power connector plate, 3D design by PantsManUK")

I did de-solder to power switch and re-soldered it in between my PicoPSU and the powerbrick. So the final result from the inside look like this:
![Re-used_Power_Switch]({{ site.baseurl }}/images/20160110/Power_Switch_Reuse.png "Reconnected the original power switch")

And from the outside it look like this:
![PSU_outside]({{ site.baseurl }}/images/20160110/picoPSU_Result.png "PSU upgrade final result outside")

----

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
----

## SATA HDD

At first I saw no need to run an HDD in my firebox. But then I figured since I've modded most of the firebox (and was in need fro some more storage) why not go all the way.  
So I went back to the forums to source some info. And sure.. this project has been done multiple times. I did notice however the successrate of this mod was not at all 100% so I figured I should take some extra time researching the components to be used for this one.  

So after some [reading](https://forum.pfsense.org/index.php?topic=105261.0) _vizi0n_'s post on the forums I decided I wanted to upgrade my firebox's capacity to >=100GB, I wanted it to run of a SATA drive instead of IDE (since those IDE's are getting rare) and it should be running en up-to-date version of pfSense.

###Parts list

- SATA drive (reused old 100GB 7K2 RPM Hitachi drive: HTS721010G9SA00)
- 44 pin (female) IDE connector (used in Dell Inspiron 5000: [eBay](http://www.ebay.com/itm/220813768916))
- 44 pin (male) to SATA adaptor (to perform IDE <> SATA: [eBay](http://www.ebay.com/itm/251926632597))
- Null modem cable as terminal BIOS access will be needed
- Laptop to perform the installation

###First steps

As with all changes.. make a backup of your current config.

Next make sure you bend or desolder the jumper on the IDE/SATA convertor as it will make contact with the firebox case if left untouched. I opted to desolder the jumper labeled JP1.

![Desolder_JP1]({{ site.baseurl }}/images/20160110/JP1.png "Desoldered jumper JP1") 

Make sure you are running at least version 7 of the firebox BIOS.

###Make it spin!

First we will be downloading the latest i386 version of pfSense, no more embedded installs from now on! I downloaded the i386, USB memstick installed with serial console.  
[https://www.pfsense.org/download/?section=downloads](https://www.pfsense.org/download/?section=downloads)

Next we will [write](https://doc.pfsense.org/index.php/Writing_Disk_Images) the memstick image to a USB key.  
After we cheked the MD5 sum of our download ofcourse! I'm not kidding.. do check the MD5sum! There is nothing more anoying then to troubleshoot an issue for hours/days only to notice it was your download which was corrupted in the first place.

To install pfSense on the HDD we will need a laptop. Slide the SATA drive in the laptop and plug in the USB key.  
This was the moment my first issue arrose. On installation I kept getting the error /usr/bin/tar -C /mnt/ -xzpf /install/pfSense.txz FAILED with returncode of 1:

![CAM_timeout]({{ site.baseurl }}/images/20160110/CAM_Error.png "CAM status: Command Timeout") 

I did check the MD5sum which was OK. Rewrote my USB key but the issue persisted..  
After a lot of googling and forum reading it was the pfSense [wiki](https://doc.pfsense.org/index.php/Boot_Troubleshooting#BIOS.2FDisk_Errors) who pointed me in the right direction.  
The key here was the AHCI BIOS settings of the laptop I used. To mitigate this error I entered te BIOS setup and configured my SATA controller mode to Compatibility instead of AHCI.

![BIOS_Setting]({{ site.baseurl }}/images/20160110/BIOS.png "Corrected SATA Controller mode in BIOS") 

Once this was done I restarted the installation from USB and it passed flawless! pfSense booted on the laptop! So far so good.

Next pull the freshly baked HDD out the laptop and assemble the parts:

![HDD_Assembly]({{ site.baseurl }}/images/20160110/HDD_Assembly.png "Assembling the HDD parts")

With the HDD in place remove you CF card from it's slot as we will no longer be needing it.

![HDD_Installed]({{ site.baseurl }}/images/20160110/HDD_Installed.png "HDD installed and CF removed")

Now it's time to make our firebox boot from it's new storage.  
Connect your null modem cable before booting as we will need to tweak some BIOS settings.  
When the memory check is running press DEL to enter the BIOS of the firebox. Once there navigate to _Standard CMOS Features_. Here you will select _IDE Channel 0 Master_:

![BIOS_HDD]({{ site.baseurl }}/images/20160110/BIOS_HDD.png "Enter HDD setup in BIOS")

In this menu you should see your drive. If this is not the case make sure both _IDE Channel 0 Master_ and _Access Mode_ are set to _Auto_ and perform _IDE HDD Auto-Detection_:

![BIOS_Channel_Master]({{ site.baseurl }}/images/20160110/BIOS_Master.png "IDE Channel 0 Master settings")

With these settings corrected navigate back to the main BIOS menu and go to _Advanced BIOS Features_ and select the first entry _Hard Disk Boot Priority_:

![BIOS_Boot_Prio]({{ site.baseurl }}/images/20160110/BIOS_Boot_Prio.png "Enter the boot prio menu in BIOS")

In this menu make sure your HDD is set as primary boot device.

![BIOS_Boot_Prio_Set]({{ site.baseurl }}/images/20160110/BIOS_Boot_Prio_Set.png "Correct boot prio settings")

When you navigate back to _Advanced BIOS Features_ you'll notice you _First Boot Device_ will be set to _Hard Disk_:

![BIOS_First_Boot]({{ site.baseurl }}/images/20160110/BIOS_First_Boot.png "First Boot Device set to HDD")

One last thing before we can get our firebox to boot!  
From the main BIOS menu navigate to _Integrated Peripherals_ and enter the sub menu _OnChip IDE Device_

![BIOS_UDMA]({{ site.baseurl }}/images/20160110/BIOS_UDMA.png "Enter the menu to correct UDMA settings")

In this menu you should set the _IDE primary Master PIO_ and _UDMA_ to _Auto_ I also enabled the _IDE HDD Block mode_ and _IDE DMA transfer access_.

![BIOS_UDMA_Set]({{ site.baseurl }}/images/20160110/BIOS_UDMA_Set.png "BIOS OnChip settings")

Now Save and exit the BIOS and let the firebox boot!

![Serial_Boot]({{ site.baseurl }}/images/20160110/Serial_Boot.png "Firebox booted from HDD")

Great success!  

----

I hope this was usefull to anyone! Have fun with your enhanced Firebox.
