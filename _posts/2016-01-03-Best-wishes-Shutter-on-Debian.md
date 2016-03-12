---
layout: post
published: true
title: A new year, a new screenshot tool
---

First of all my best wishes to all of you!

Today I needed to create a small how to document for someone, explaining how to update the core and modules of their Drupal site. This needed to be a fool-proof document. So naturally a lot of screenshots were required.

I started off with an empty writer file in my Libre Office and launched my FileZilla client (which by the way is [available](https://packages.qa.debian.org/f/filezilla/news/20151222T163906Z.html) again via the Debian testing repo's since 22/12/2015). 
After connecting to the ftp server I took my first screenshot. By default Gnome is able to make [screenshots](https://wiki.debian.org/ScreenShots) and copies them to your clipboard or saves them in a folder. This required me to launch Gimp in order to crop the image and add the neccessary arrows etc. 
Hmmm... bummer, this would be to time consuming for something very trivial...

At work (on my Windows 7 machine) I got used to the built in [snipping tool](https://en.wikipedia.org/wiki/Snipping_Tool) provided my Microsoft. The tool lets you quickly select an area to capture, edit that capture and copy it to the clipboard or save it somewhere. So you can image I didn't want to continue before upgrading my machine's default screenshot tool to something more user friendly.

After some googeling I stumbled upon this tool: [Shutter](http://shutter-project.org/)

This seems to do everything (and more?) I need from a screenshot tool!
So I launched my terminal and installed shutter from the testing repo's

```
sudo apt-get install shutter
```

Next I launched shutter and, once it was done loading it's (perl) modules, opened the preferences to check out it's capabilities (alt+p). The only thing I modified for now was the **Save** settings so my screenshots aren't automatically saved.
![Shutter preferences]({{ site.baseurl }}/images/20160103/shutter-preferences.png "Shutter Preferences")

While scrolling through the preferences I noticed that shutter has some other cool features! 
For example you can make it upload you screencaptures to things like an FTP server, Dropbox, etc. It also supports plugins. By default it comes shipped with some effect plugins like a sepia- and barrel distortion filter. 

Anyways, enough wandering off for now!

I reopened my FileZilla client, took my screenshot again using shutter and opened it with the built-in editing tool. Guess what... Shutter comes with a very nice built-in editor. It includes the standard things you would expect to see like a highlighter, line drawer, rectangles, etc. But by default it also includes an arrow drawing tool and a tool to pixelize areas of the screenshot to hide private data etc. Now I want this in my Windows snipping tool at work as well!

Half way through the document I figured I'd better replace the default Gnome screenshot tool by shutter completely. I think the developers of shutter figured this would be something people would naturally be wanting to do. They created an article on [how to do this](http://shutter-project.org/faq-help/set-shutter-as-the-default-screenshot-tool/#gnome) in their FAQ section.

I added two links for now:

- Screenshot Area-Select bound to **PrtSc**-button
    - `shutter --select --delay=0 -c`
        - *--select*    lets you select the area you want to capture
        - *--delay=0*   Sets the delay to 0 seconds
        - *-c*          includes your cursor in the screenshot
        - Screenshot Window Select
    - `shutter -w --delay=0 -c`
        - *-w*          makes you select the windows you want to capture
        - *--delay=0*   sets the delay to 0 seconds
        - *-c*          includes your cursor in the screenshot
- For more options check out the shutter man pages
    - `man shutter`

Than I wondered if there was a way to disable the other default screenshot keybindings in Gnome. And guess what... there is! Don't you just love Linux :)

Open the *system settings* and go to keyboard
![Select keyboard from system settings menu]({{ site.baseurl }}/images/20160103/keyboard-system-settings.png "Select keyboard from system settings menu")

Under the *Shortcuts* tab select the *Screenshots* submenu. Once there select the line(s) and hit backspace to disable these keybindings
![Select the line(s) and hit backspace to disable]({{ site.baseurl }}/images/20160103/disable-screenshots-keybindings.png "Select the line(s) and hit backspace to disable")

Hope I've helped, have fun with Shutter!
