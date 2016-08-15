---
layout: post
published: true
title: Installing ESXi 6.0 with less then 4GB RAM

tags:
- VMware
- ESXi
---

Today I figured it would be nice to repurpose my scrap laptop. It's basically an old Lenovo of which the screen was broken and removed so it's headless (literally). Apart from that it still runs fine, has a Core i5 under the hood and supports up to 8GB RAM. At the moment it's running with just 4GB though.. Which, apparently, poses a problem when you try to install ESXi 6.0. While all the hardware is discovered correctly during install I received a MEMORY_SIZE ERROR during install.

So here is how to circumvent this error:

During install, when you receive the MOMORY_SIZE ERROR press ALT+F1 to you can get shell access.

Next we log in as root with a blank password.
Once we're logged in as root perform the following commands:
```
cd /usr/lib/vmware/weasel/util
rm upgrade_precheck.pyc
mv upgrade_precheck.py upgrade_precheck.py.BAK
cp upgrade_precheck.py.BAK upgrade_precheck.py
chmod 666 upgrade_precheck.py
vi upgrade_precheck.py 
# search for MEM_MIN_SIZE and find (4*1024) * SIZE_MiB. Replace 4* with 2* and :wq.
```

Now kill the weasel process. (find it's PID and kill it)
```
ps -c | grep weasel
# write down the PID

kill -9 <PID>
# The installer will automagically restart and should pass the memory test now
```

Have fun running your ESXi 6.0 sub 4GB RAM!
