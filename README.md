**WARNING: This is a work in progress as of 3.4.2019. Don't use it yet, until the version number below reaches 1.0**

# odroid-xu4-setup
**VERSION 0.2**

How to set up an ODroid XU4 with Kodi, Mame and an external USB drive

I describe all the steps to install Ubuntu Linux on an ODroid XU4 and use it as a media and game center using Kodi and Mame.
Installing Mame is optional as well as configuring the USB drive, the joystick, etc... You can stop at the Kodi level or go further and have a fully functional video, music and game machine

##### Table of contents
1. [Install Linux and Kodi on your ODroid XU4](#Install-Linux-and-Kodi-on-your-ODroid-XU4)<br/>
2. [Install an USB external drive on your ODroid XU4](#Install-an-USB-external-drive-on-your-ODroid-XU4)<br/>
3. [Install MAME to play arcade games from Kodi](#Install-MAME-to-play-arcade-games-from-Kodi)<br/>

## Install Linux and Kodi on your ODroid XU4

To install all of this, follow the steps:

1. The first step is ideally done on another Linux box. If you don't have one yet (what ???), better to migrate to Linux as soon as you can

	Download an image from https://wiki.odroid.com/odroid-xu4/os_images/linux/ubuntu_4.14/20181203

2. Check your image is valid with `md5sum`

	and compare the result with the corresponding m5sum file you can find here: https://odroid.in/ubuntu_18.04lts/XU3_XU4_MC1_HC1_HC2/

	```bash
	md5sum	ubuntu-18.04.1-4.14-mate-odroid-xu4-20181203.img.xz
	```

3. Assuming you're doing all of this on a Linux box, decompress the file you downloaded:

	```sh
	xz -d ubuntu-18.04.1-4.14-mate-odroid-xu4-20181203.img.xz
	```

4. Plugin your SD card on an USB port of your Linux box
5. Search for the device with `lsblk`

	```bash
	lsblk
	```

	**WARNING**

	Let's assume your SD card is on /dev/sdX. Be very careful to choose the correct device or you will risk to erase the content of another disk.

	**WARNING**

6. Write the image to the SD card

	```bash
	sudo dd if=ubuntu-18.04.1-4.14-mate-odroid-xu4-20181203.img.xz of=/dev/sdX bs=1M conv=fsync
	sudo sync
	```

7. Umount your SD card either from your desktop environment or on the command line:

	```bash
	sudo umount /dev/sdX
	```

8. Insert your SD card in your odroid XU4 and boot it up

	Then login on the desktop with the credentials as given at https://wiki.odroid.com/odroid-xu4/os_images/linux/ubuntu_4.14/20181203#access_credentials

	At this stage you have a fully functional ODroid XU4 desktop. The next step will be to install Kodi and remove this direct desktop access so that your box will directly boot on the Kodi interface more like a regular TV set

10. Open a terminal (search in the menu on the top-left of the screen)

11. Update the packages

	```bash
	sudo apt update
	sudo apt upgrade
	sudo apt dist-upgrade
	```

	During the update, the first time you boot your machine, you may get an error regarding a lock file with dpkg (/var/lib/dpkg/lock), it means that one of the automated update procedure is still running which is very likely with a new install. Just let it run until the end and try the commands again.

	During the update, if you get a message about boot.ini being replaced, just say 'OK'

	During the update, if you get a question about /etc/apt-fast.conf, answer Y (for Yes)

14. The good news is that kodi installed by default. Kodi is installed by default. But if by any chance it is not (?), you can install it with:

	```bash
	sudo apt install kodi kodi-bin kodi-data
	```

15. The Mali graphical driver is installed by default and works well. Go in a terminal and run
	`glmark2-es2` and you should see a demo (a horse). FPS should be around 300 frames/second
	run `glmark2` and you should see the same demo. FPS should be around 30 frames/second. Terrible! The reason is because the Mali driver only supports OpenGL ES (Embedded System) and not plain OpenGL. The solution is to use a library called [gl4es](https://github.com/ptitSeb/gl4es), which you can find on Github.

16. Install gl4es

	In a terminal, install the following development tools:

	```bash
	sudo apt install git cmake xorg-dev gcc g++ build-essential
	```

	Get the source code of gl4es. So, gl4es is an implementation of OpenGL using ... OpenGL ES (instead of implementing the library directly). So for systems like ODroid XU4 with a driver supporting only OpenGL ES, it is possible to use software based on OpenGL too with hardware acceleration:

	```bash
	git clone https://github.com/ptitSeb/gl4es.git
	cd gl4es
	```

	And compile it:
	```bash
	mkdir build
	cd build
	cmake .. -DODROID=1
	make
	```

	It will compile and display its progress with a lot of messages. Toward the end, if the compilation has been successfull, you should see the following:
	```
	[ 96%] Building C object src/CMakeFiles/GL.dir/glx/utils.c.o
	[ 98%] Linking C shared library ../lib/libGL.so.1
	[100%] Built target 
	```

	Save the Mesa version of OpenGL (the one installed by default):

	```bash
	sudo mv /usr/lib/arm-linux-gnueabihf/libGL.so.1 /usr/lib/arm-linux-gnueabihf/save.libGL.so.1
	sudo mv /usr/lib/arm-linux-gnueabihf/libGL.so.1.0.0 /usr/lib/arm-linux-gnueabihf/save.libGL.so.1.0.0
	```

	and copy your new compiled version:
	```bash
	sudo cp lib/libGL.so.1 /usr/lib/arm-linux-gnueabihf/
	sudo ldconfig
	```

	Now check that everything is fine by running:
	```bash
	glxinfo|head -6
	```

	If you see the following, then you're good to go:
	```
	LIBGL: Initialising gl4es
	LIBGL: v1.1.1 built on Mar 28 2019 06:22:26
	LIBGL: Using GLES 2.0 backend
	LIBGL: loaded: libGLESv2.so
	LIBGL: loaded: libEGL.so
	LIBGL: Using GLES 2.0 backend
	```
	(of course the date on the second line will be different for you)

	Now you can run glmark2 again:
	```bash
	glmark2
	```

	and this time you should see FPS around 300.

	At this stage, your ODroid XU4 also have full OpenGL support. You can even use software like [Blender](https://www.blender.org/).

19. For security reason, kodi and its associated programs will be run by a user with limited privileges, with no password and automatic login. We will call the user `kodi`:

	```bash
	sudo adduser --disabled-password --gecos "" kodi
	```

	The output will be:
	```
	Adding user `kodi' ...
	Adding new group `kodi' (1000) ...
	Adding new user `kodi' (1000) with group `kodi' ...
	Creating home directory `/home/kodi' ...
	Copying files from `/etc/skel' ...
	```

20. Assign some privileges to this user:

	```bash
	usermod -a -G cdrom,video,plugdev,users,dialout,dip,input,netdev,audio,pulse kodi
	```

21. Add options to allow Kodi to start its own X server:

	```bash
	sudo sed -ie 's/allowed_users=console/allowed_users=anybody/g' /etc/X11/Xwrapper.config
	sudo sed -ie "\$aneeds_root_rights = yes" /etc/X11/Xwrapper.config
	```

22. Add the Kodi service to `systemd` (better to copy and paste the following rather than typing it):

	```bash
	sudo cat > /etc/systemd/system/kodi.service <<EOF
	[Unit]
	Description = Kodi Media Center

	After = systemd-user-sessions.service network.target sound.target mysql.service
	Wants = mysql.service

	[Service]
	User = kodi
	Group = kodi
	Type = simple
	#PAMName = login # you might want to try this one, did not work on all systems
	ExecStart = /usr/bin/xinit /usr/bin/dbus-launch --exit-with-session /usr/bin/kodi-standalone -- :0 -nolisten tcp vt7
	Restart = on-abort
	RestartSec = 5

	[Install]
	WantedBy = multi-user.target
	EOF
	```

23. Enable Kodi at boot time:

	```bash
	sudo systemctl enable kodi
	sudo systemctl set-default multi-user.target
	```

24.	**From now on you have a fully functional Kodi system**. If you want to reboot, your ODroid XU4 will directly start on Kodi. I you want to add more features, like [MAME](https://www.mamedev.org/) ([Mame on Wikipedia](https://en.wikipedia.org/wiki/MAME)), an external USB drive, a joystick, a firewall, you can follow the rest.

	If you want to stop and enjoy your ODroid XU4 now, you can reboot it:

	```bash
	sudo reboot
	```

In the next sections, I'll show you how to:
- install an external USB drive and have it automounted (for Kodi or Mame for example),
- install and configure Mame and use it from Kodi
- install and configure a joystick for Mame
- remove software and services which are not necessary and use memory and CPU cycle for nothing
- add firewall rules to set up the web connection for Kodi and use one of the many Android/iOS apps to control Kodi
- create a swapfile to add extra virtual memory to your system

Everything can be done either by connecting to your ODroid XU4 with `ssh` or on the screen directly (you'll need a keyboard). If you want to connect on the screen directly, after rebooting your ODroid XU4 on Kodi, you can press `Ctrl+Alt+F1` to switch to a terminal (text screen). At any time, you can go back to Kodi by pressing `Alt+F7`.

## Install an USB external drive on your ODroid XU4


1. Switch to a terminal or login with `ssh`.

2. Install `usbmount` to automount USB drives:

	```bash
	sudo apt install usbmount
	```

	There is a bug in the version 0.0.22 of `usbmount` as provided by the stock Ubuntu Linux when connecting 2 USB drives at the same time. The bug has been fixed with `usbmount 0.0.24` which is not yet on the Ubuntu repository. You can upgrade it manually, if you like:

	If your second drive (or even the first) doesn't mount properly when you plug it in, you can try to upgrade `usbmount` as follows:

	```bash
	git clone https://github.com/rbrito/usbmount.git
	cd usbmount
	sudo apt-get update && sudo apt-get install -y debhelper build-essential
	dpkg-buildpackage -us -uc -b
	cd ..
	sudo dpkg -i usbmount_0.0.24_all.deb
	```

## Install MAME to play arcade games from Kodi

This section is long and will require external resources if you want to have all the bells and whistles of a full-featured MAME installation. From time to time, we will refer to external guides too.

1. Install MAME

	```bash
	sudo apt install mame mame-data mame-doc mame-extra mame-tools
	```

	Change the mame rendingir driver to OpenGL ES.
	For some reasons, the OpenGL driver (through `gl4es`) didn't work. If someone has a better solution, please contact me and share.

	```bash
	sudo sed -i 's/opengl/opengles/' /etc/mame/mame.ini
	```

28. Install Advanced Mame Launcher plugin from Kodi

	We're going to follow the official guide of a Kodi plugin called [Advanced Mame Launcher (AML)](https://forum.kodi.tv/showthread.php?tid=304186) and adapt a few steps to follow our setup. Here, I assume that we have an external USB drive connected to the Odroid XU4. It will be helpful to store data for MAME.

	1. Assuming you're on a text terminal (see above on how to switch to a text terminal from Kodi), the first step is to create directories to store MAME data. The external USB drive is mounted to `/media/usb0` and by default to `/media/usb` too. We will assume it's the case from now on.

	2. We now create the directory structure as required by AML for MAME:

	```bash
	cd /media/usb
	sudo mkdir mame
	sudo chown kodi.kodi mame
	sudo su - kodi
	cd /media/usb/mame
	mkdir -p AML-ROMs AML-asset AML-CHDs AML-SL-ROMs AML-SL-CHDs AML-assets/samples/
	cd AML-assets
	mkdir artpreviews artwork cabinets clearlogos covers_SL cpanels fanarts fanarts_SL flyers manuals manuals_SL marquees PCBs snaps snaps_SL titles titles_SL videosnaps videosnaps_SL
	cd
	ln -s /media/usb/mame/* .
	exit
	```
	
	2. Fill in the directories with the latest data for Mame as described in https://forum.kodi.tv/showthread.php?tid=304186:

	```bash
	sudo su - kodi
	cd /media/usb/mame/AML-assets
	
	wget http://www.progettosnaps.net/catver/ -q -O - | grep 'download?tipo=catver' | sed "s#.*href=\"\(.*\)\".*#wget -q 'http://www.progettosnaps.net\1' -O file.zip#"|sh
	unzip -jq file.zip *.ini
	rm file.zip

	exit
	```


Next we need to modify mame.ini to reflect this directories' structure:

sudo sed -i 's#^rompath \+.*$#rompath /home/kodi/AML-ROMs/#' /etc/mame/mame.ini
sudo sed -i 's#^samplepath \+.*$#samplepath /home/kodi/AML-assets/samples/#' /etc/mame/mame.ini

	You can edit the file instead or re-use the same 'sed' kind of lines to keep modifying mame.ini if you add more resources. You can download various .ini files with information about the games in Mame. Read the same installation guide. Once you've put everything you want, you can install the Kodi plug-in called Advanced Mame Launcher (AML).
	 Again, from https://forum.kodi.tv/showthread.php?tid=304186, you can follow the paragraph called Setting up MAME assets and Software List assets to add more resources.
	 Then follow the installation paragraph called Setting up Advanced MAME Launcher (Easy mode)
	 And that's it. Mame is ready from Kodi

29. Install joystick support
sudo apt install joystick jstest-gtk

30. Calibrate the joystick
As most of Mame games will require a simple joystick with buttons, calibration will be very simple
You can use jstest-gtk but as we already install Kodi, we will do the calibration from the command line only. With click joystick, the only calibration is to associate buttons (inside the joystick for the directions and fire/select buttons), with their respective direction or function. The calibration will be available system-wide and therefore will be used by mame


31. Create a swap file to add more virtual memory
In order to _extend_ the memory of the Odroid XU4, we want to add a swap file. However, it's not a good idea to use the SD-card. It's faster but swap file can be written over and over a lot of time, thus reducing the life of the card. So we will use the external USB drive, connected as described above (where we assume the drive is always connected and mounted at /media/usb0):

	sudo fallocate -l 2G /media/usb0/swapfile
	sudo chmod 600 /media/usb0/swapfile
	sudo mkswap /media/usb0/swapfile
	sudo swapon /media/usb0/swapfile
	sudo sed -ie "\$a/media/usb0/swapfile swap swap defaults 0 0" /etc/fstab
