<!--
    Windows-Persistent-Ramdisk (WPR) is a config repo used to create a ramdisk on Windows.
    Copyright (C) 2017  Rémi Ducceschi (remileduc) <remi.ducceschi@gmail.com>

    This program is free software: you can redistribute it and/or modify
    it under the terms of the GNU General Public License as published by
    the Free Software Foundation, either version 3 of the License, or
    (at your option) any later version.

    This program is distributed in the hope that it will be useful,
    but WITHOUT ANY WARRANTY; without even the implied warranty of
    MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
    GNU General Public License for more details.

    You should have received a copy of the GNU General Public License
    along with this program. If not, see <http://www.gnu.org/licenses/>.
-->

[![License GPL v3.0](https://img.shields.io/badge/license-GNU%20GPL%20v3.0-blue.svg)](https://github.com/remileduc/windows-persistent-ramdisk/blob/master/LICENSE)

windows-persistent-ramdisk
======================

Create a ramdisk for tmp and cache folders with persistent saves amoung reboots - for Windows.


This repo creates a ramdisk in Windows and show you how to store temporary data in it, along with Firefox's cache. But as long as the ramdisk is created, you can store whatever cache you want in it.

The storage of temporary files has a big advantage in Windows: all these files are removed on reboot, which is not the case by default and so you lose a lot of space for nothing. However, for big updates, it seems that Windows expect some files to still be in the temporary folder after reboot, which will not be the case with a ramdisk, causing the failure of the update. 2 solutions:
- you don't store system temporary files in the ramdisk
- you disable the ramdisk for the update and reenable it afterwards.

As Windows doesn't support ramdisk by default, we need to use another (free and open-source) software: ImDisk. You can download the last version here: http://www.ltr-data.se/opencode.html/#ImDisk

Unfortunately, this means that, unlike on Linux, the cache folders will only be copied after you opened your session. This doesn't matter for Firefox (in the worst case, the cache will be filled when Firefox is running).

Also, in the non-professional versions of Windows, it is not possible to run a script on shutdown. The scripts shown here save the persistent files every hour and on lock / end of session, but not on shutdown. This means that in the worst case, you will lose 1 hour of cache for Firefox, which is not critical.

I use these scripts for my own, and as you can see, it may be improved. Feel free to contribute!

Prerequisites
=============

You need to install ImDisk. You can download it from here: http://www.ltr-data.se/opencode.html/#ImDisk

Once installed, you can continue to next section.

Create a ramdisk
================

Here, we will see how to automate the creation of a ramdisk at boot, along with the save of persistent files like cache.

Download the scripts
--------------------

First, you need to download all the scripts and save them in `C:\myscripts\`. You can store it in another folder, but then you'll need to change the scripts. In this folder, you should have at least the following files:
- `boot_ramdisk.bat`: script to create and fill the ramdisk
- `boot_ramdisk_save_persistent.bat`: script to save the persistent files from the ramdisk to a drive
- `launcher.vbs`: to lauch the scripts silently (without creating a console popup)
- `RamDisk.xml`: Windows task to create the ramdisk
- `save_ramdisk.xml`: Windows task to save persistent files every hours
- `save_ramdisk_shutdown.xml`: Windows task to save persistent files on lock and session logout

You also need to create a folder on your HDD/SSD where persistent files will be stored so you won't lose them amoung reboots. Currently, it is in `D:\ramdisk\persistent`. If you want to change, simply change the path in the 2 `.bat` files.

By default, a ramdisk of 3 Gio is created. If you want to change the size, you need to edit the file `boot_ramdisk.bat` and change the size given (`3G` in the example):

```bash
	imdisk -a -s 3G -m Z: -p "/fs:ntfs /q /y"
```

Import the tasks
----------------

Open the task scheduler (or run `taskschd.msc`). From the library, import the 3 tasks:
	- `C:\myscripts\RamDisk.xml`
	- `C:\myscripts\save_ramdisk.xml`
	- `C:\myscripts\save_ramdisk_shutdown.xml`

You can now launch the task `RamDisk` and check that a `Z:` drive has been created, with a size of 3 Gio by default.

Temporary files on ramdisk
==========================

This is very easy. You just need to change the environment variables `TMP` and `TEMP`. To do so, follow this howto: https://www.computerhope.com/issues/ch000549.htm

You have 2 sections: User's variables and system ones. You can safely change the user's variables: change the `TMP` and `TEMP` variables to both point to `Z:\%USERNAME%\TEMP`.

If you change the system variables too, you may experiment some troubles with Windows Update, see the really first section. If you still want to do it, make the system `TMP` and `TEMP` variables point to `Z:\system\TEMP`.

That's it, you can reboot and check the changes. You can also remove all the files from `C:\Users\%USERNAME%\AppData\Local\Temp`, and from `C:\Windows\Temp` if you changed the system temporary directory too.

Put Firefox's cache on ramdisk
==============================

In Firefox, type `about:config` in the URL bar. Then, create or modify the key `browser.cache.disk.parent_directory` and put the value `Z:\\%USERNAME%\\persistent\\firefox\\cache`.

Reboot Firefox. Now the cache will be in the ramdisk, which gives you a faster Firefox and a longer live to your SSD. If you don't want to lose your current cache, you can copy the folder `C:\Users\%USERNAME%\AppData\Local\Mozilla\Firefox\Profiles\XXXXXXXX.default\cache2` in `Z:\%USERNAME%\persistent\firefox\cache`.
