---
layout: post
published: true
title: A new year, a new screenschot tool
---

First of all my best wishes to all of you!

Today I needed to create a small how to document for someone, explaining how to update the core and modules of their Drupal site. This needed to be a fool-proof document so naturally a lot of screenshots were required.

So I created an empty writer file in my Libre Office and launched my FileZilla client (which by the way is [available](https://packages.qa.debian.org/f/filezilla/news/20151222T163906Z.html) again via the Debian testing repo's since 22/12/2015). After connecting to my ftp server I took my first screenshot. By default Gnome is able to make [screenshots](https://wiki.debian.org/ScreenShots) and copy them to your clipboard or save them in a folder. So now this required me to launch Gimp in order to crop the image and add the neccessary arrows etc. hmm... bummer, this would be very time consuming for somthing very trivial...

At work (on my Windows 7 machine) I got used to the built in [snipping tool](https://en.wikipedia.org/wiki/Snipping_Tool) provided my Microsoft. The tool lets you quickly select an area to capture, edit the captured area and copy it to clipboard or save it somewhere. So you can image I didn't want to continue before upgrading my machine's default screenshot tool to something more user friendly.

Next I stumbled upon this tool: [Shutter](http://shutter-project.org/)
This seems to do everything I need from a screenshot tool!

So I launched my terminal and installed shutter from the testing repo's
```bash
sudo apt-get install shutter
```

Next I launched shutter and, once it was done loading it's (perl) modules, opened the preferences to check out it's capabilities (alt+p). The only thing I modified for now was the **Save** settings so my screenshots aren't automatically saved.
![Shutter preferences][/images/20160103/shutter-preferences.png]

While scrolling through the preferences I noticed that shutter has some other cool features! 
For example you can make it upload you screencaptures to things like an FTP server, Dropbox, etc. It also supports plugins, by default it comes shipped with some effect plugins like a sepia- and barrel distortion filter. 

Anyways, enough wandering off for now!
I re-opened my FileZilla client and took my screenshots again using shutter and opened it with the built-in editing tool and my god... Shutter comes with a very nice built-in editor. It includes the standard things you would expect to see like a highlighter, line drawer, rectangles, etc. But by default it also includes an arrow drawing tool and a tool to pixelize areas of the screenshot to hide private data etc. Very sweet added value right there!

Once my document was finished I figured I'd better replace the default Gnome screenshot tool by shutter completely. I think the developpers of shutter figured this would be something people would naturally be wanting to do since they created an article on [how to do this](http://shutter-project.org/faq-help/set-shutter-as-the-default-screenshot-tool/#gnome) in their FAQ section.

I added two links for now:
- Screenshot Area-Select bound to **PrtSc**-button
	- `shutter --select --delay=0 -c`
		- *--select* lets you select the area you want to capture
		- *--delay=0* sets the delay to 0 seconds
		- *-c* includes your cursor in the screenshot
- Screenshot Window Select
	- `shutter -w --delay=0 -c`
		- *-w* makes you select the windows you want to capture
		- *--delay=0* sets the delay to 0 seconds
		- *-c* includes your cursor in the screenshot
- For more options check out the shutter man pages
	- `man shutter`

Than I wondered if there was a way to disable the other default screenshot keybindings in Gnome. And guess what... there is! don't you just love Linux :)

Open the *system settings* and go to keyboard
![Select keyboard from system settings menu][/images/20160103/keyboard-system-settings.png]

Under the *Shortcuts* tab select the *Screenshots* submenu. Once there selec the line(s) and hit backspace to disable these keybindings
![Select the line(s) and hit backspace to disable][/images/20160103/disable-screenshots-keybindings.png]
