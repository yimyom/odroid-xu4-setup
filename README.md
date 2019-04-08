**WARNING: This is a work in progress as of 3.4.2019. Don't use it yet, until the version number below reaches 1.0**

# odroid-xu4-setup
**VERSION 0.8**

How to set up an Odroid XU4 with Kodi, Mame and an external USB drive

I describe all the steps to install Ubuntu Linux on an Odroid XU4 and use it as a media and game center using Kodi and Mame.
Installing Mame is optional as well as configuring the USB drive, the joystick, etc... You can stop at the Kodi level or go further and have a fully functional video, music and game machine

##### Table of contents
1. [Install Linux and Kodi on your Odroid XU4](#Install-Linux-and-Kodi-on-your-Odroid-XU4)<br/>
2. [Install an USB external drive on your Odroid XU4](#Install-an-USB-external-drive-on-your-Odroid-XU4)<br/>
3. [Install MAME to play arcade games from Kodi](#Install-MAME-to-play-arcade-games-from-Kodi)<br/>
4. [Install a joystick to play with MAME](#Install-a-joystick-to-play-with-MAME)<br/>
5. [Remove unused software to save memory and CPU cycle](#Remove-unused-software-to-save-memory-and-CPU-cycle)<br/>

## Install Linux and Kodi on your Odroid XU4

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

	At this stage you have a fully functional Odroid XU4 desktop. The next step will be to install Kodi and remove this direct desktop access so that your box will directly boot on the Kodi interface more like a regular TV set

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

	Get the source code of gl4es. So, gl4es is an implementation of OpenGL using ... OpenGL ES (instead of implementing the library directly). So for systems like Odroid XU4 with a driver supporting only OpenGL ES, it is possible to use software based on OpenGL too with hardware acceleration:

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

	At this stage, your Odroid XU4 also have full OpenGL support. You can even use software like [Blender](https://www.blender.org/).

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

24.	**From now on you have a fully functional Kodi system**. If you want to reboot, your Odroid XU4 will directly start on Kodi. I you want to add more features, like [MAME](https://www.mamedev.org/) ([Mame on Wikipedia](https://en.wikipedia.org/wiki/MAME)), an external USB drive, a joystick, a firewall, you can follow the rest.

	If you want to stop and enjoy your Odroid XU4 now, you can reboot it:

	```bash
	sudo reboot
	```

In the next sections, I'll show you how to:
- install an external USB drive and have it automounted (for Kodi or Mame for example),
- install and configure Mame and use it from Kodi
- install and configure a joystick for Mame
- remove software and services which are not necessary and use memory and CPU cycle for nothing

Everything can be done either by connecting to your Odroid XU4 with `ssh` or on the screen directly (you'll need a keyboard). If you want to connect on the screen directly, after rebooting your Odroid XU4 on Kodi, you can press `Ctrl+Alt+F1` to switch to a terminal (text screen). At any time, you can go back to Kodi by pressing `Alt+F7`.

## Install an USB external drive on your Odroid XU4


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
	mkdir -p AML-ROMs AML-CHDs AML-SL-ROMs AML-SL-CHDs AML-assets/samples/
	cd AML-assets
	mkdir artpreviews artwork cabinets clearlogos covers_SL cpanels fanarts fanarts_SL flyers manuals manuals_SL marquees PCBs snaps snaps_SL titles titles_SL videosnaps videosnaps_SL
	cd
	ln -s /media/usb/mame/* .
	exit
	```
	
	2. Fill in the directories with the latest data for Mame as described in https://forum.kodi.tv/showthread.php?tid=304186. Again, better to copy and paste the script rather than copying manually

	```bash
	sudo su - kodi
	cd /media/usb/mame/AML-assets
	# catlist.ini catver.ini genre.ini genre_OWS.ini mature.ini not_mature.ini
	wget http://www.progettosnaps.net/catver/ -q -O - | grep 'download?tipo=catver' | sed "s#.*href=\"\(.*\.zip\)\".*#wget -q 'http://www.progettosnaps.net\1' -O file.zip#"|sh
	unzip -jq file.zip '*.ini'
	rm file.zip

	# nplayers.ini
	wget http://nplayers.arcadebelgium.be/ -q -O - | grep -E 'nplayers[[:digit:]]{4}\.zip' | sed "s#.*href=\"\(http://nplayers.*zip\)\">.*#wget -q '\1' -O file.zip#" | sh
	unzip -jq file.zip nplayers.ini
	rm file.zip

	# bestgames.ini
	wget http://www.progettosnaps.net/bestgames/ -q -O - | grep 'download?tipo=bestgames' | sed "s#.*href=\"\(.*\.zip\)\".*#wget -q 'http://www.progettosnaps.net\1' -O file.zip#"|sh
	unzip -jq file.zip '*.ini'
	rm file.zip

	# series.ini
	wget http://www.progettosnaps.net/series/ -q -O - | grep 'download?tipo=series' | sed "s#.*href=\"\(.*\.zip\)\".*#wget -q 'http://www.progettosnaps.net\1' -O file.zip#"|sh
	unzip -jq file.zip '*.ini'
	rm file.zip

	# history.dat
	wget https://www.arcade-history.com/?page=download -q -O - | grep -o 'href="[^"]*\.zip"' | sed 's#href=\"\.\.\(.*zip\)\"#wget https://www.arcade-history.com\1 -q -O file.zip#'|sh
	unzip -jq file.zip history.dat

	# mameinfo.dat
	wget 'http://mameinfo.mameworld.info' --header="User-Agent: Firefox/70.0" -q -O - |grep -o 'href=\"[^"]*Mameinfo.*\.zip"'|sort|tail -1| sed 's#href=\"\(.*zip\)\"#wget --header=\"User-Agent: Firefox/70.0\" \1 -q -O file.zip#'|sh
	unzip -qjp file.zip *.7z > mameinfo.7z
	7z e '-i!mameinfo.dat' mameinfo.7z > /dev/null
	rm file.zip mameinfo.7z

	# gameinit.dat
	wget http://www.progettosnaps.net/gameinit/ -q -O - | grep 'download?tipo=gameinit' | sed "s#.*href=\"\(.*\.zip\)\".*#wget -q 'http://www.progettosnaps.net\1' -O file.zip#"|sh
	unzip -jq file.zip english/gameinit.dat
	rm file.zip

	# command.dat
	wget http://www.progettosnaps.net/command/ -q -O - | grep 'download?tipo=command' | sed "s#.*href=\"\(.*\.zip\)\".*#wget -q 'http://www.progettosnaps.net\1' -O file.zip#"|sh
	unzip -jq file.zip Longhand/command.dat
	rm file.zip

	exit
	```
29.  Next we need to modify `mame.ini` to reflect this directories' structure:

```bash
sudo sed -i 's#^rompath \+.*$#rompath /home/kodi/AML-ROMs/#' /etc/mame/mame.ini
sudo sed -i 's#^samplepath \+.*$#samplepath /home/kodi/AML-assets/samples/#' /etc/mame/mame.ini
```

In https://forum.kodi.tv/showthread.php?tid=304186, you can follow the paragraph called _Setting up MAME assets and Software List assets_ to add more resources and assets. This is the way to get extra pictures, logos, etc...
You can read the page here: http://forum.pleasuredome.org.uk/index.php?showtopic=30715 about the MAME Extra packages to understand all the type of resources you can find on the Net for Mame.

30. Next step is to follow (again) the guide at https://forum.kodi.tv/showthread.php?tid=304186 in the paragraph _Setting up Advanced MAME Launcher (Easy mode)_
	To do that, you have to `exit` from the text terminal you're connected too. Type in `exit` or use the combination `Ctrl+D`. Then go back to Kodi's screen by hitting `Alt+F7`.
	When in Kodi, follow the paragraph mentionned above.

	And Mame is ready !

## Install a joystick to play with MAME
Ideally, playing with MAME requires a nice joystick. Here are two examples of joystick I've built myself. It's a good exercise of woodwork, painting, designing and electronics and a fun game for the family. I've made them with planks I collected from a construction site nearby. Good for the environment to recycle things too.

![joystick1](/images/joystick1.png) ![joystick2](/images/joystick2.png)

1. Install the software to support and calibrate the joystick. Arcade joysticks are easy to build or can be bought on Internet. They work very well and ideal for playing with MAME's arcade games.
	```bash
	sudo apt install joystick jstest-gtk
	```

2. Calibrate the joystick
	As most of Mame games will require a simple joystick with buttons, calibration will be very simple.
	You can use `jstest-gtk` but as we already install Kodi, we will do the calibration from the command line only. With click joystick, the only calibration is to associate buttons (inside the joystick for the directions and fire/select buttons), with their respective direction or function. The calibration will be available system-wide and therefore will be used by mame
	- if you can plug your joystick to your Linux machine, I recommend to use, in first instance, a small programm called `jstest-gtk`. It's a simple GUI and you can check what's the proper direction of your joystick. In my case, I use a DragonRise compatible joystick (the one on the picture), with 4 connectors for up, down, left and right. But there are a few problems which we will fix with the calibration. First of all, the left-right and up-down are inversed and then the up-down axis is upside down. So to make it short: up is right, down is left, right is down and left if up!!! I can see that on the `jstest-gtk` interface.
	- another option (since Kodi 17) is to setup your joystick directly from Kodi. Read the tutorial here: https://kodi.wiki/view/HOW-TO:Configure_controllers

	As there are many models of joystick, I won't cover all the possible configurations but please contribute and I'll add your solution to this guide.


## Remove unused software to save memory and CPU cycle

This section is not mandatory but you can find it useful to make your Odroid XU4 lighter and more responsive. The XU4 has 2Gb of memory, which (at the time of writing) is good for a Single Board Computer (most of them have 0.5 to 1Gb, but I can see some 2 to 4 Gb finally coming on the market). So memory is a precious resource as well as CPU cycle. You don't want any slowdown of your machine while watching a movie or playing a game.
I selected a few services which I think are not necessary for a Kodi/Mame installation. Here is the way to stop and disable them. However, we will not uninstall the software, so that you can enable them again, should your needs change in the future.

1. Cups the printing server
	As you don't really need to print from Kodi or Mame, it's safe to remove the print server (named CUPS):
	```bash
	sudo systemctl stop cups
	sudo systemctl stop cups-browsed
	sudo systemctl disable cups
	sudo systemctl disable cups-browsed
	```

	To re-enable it
	```bash
	sudo systemctl enable cups
	sudo systemctl start cups
	```

2. UPower controls the power source. It's useful in a smartphone, a laptop or an embedded system. However, in the case of a Kodi/Mame entertainment system in your living room, the only power source is the socket wall and your Odroid XU4 is supposed to be connected all the time. So you can safely remove this one too:
	```bash
	sudo systemctl stop upower
	sudo systemctl disable upower
	```

3. Whoopsie is the error reporting daemon for Ubuntu, mainly used by desktop environment when something crashes. As we're only using Kodi, it's not necessary here:
	```bash
	sudo systemctl stop whoopsie 
	sudo systemctl disable whoopsie 
	```

4. ModemManager is a daemon which controls mobile broadband (2G/3G/4G) devices and connections. The Odroid XU4 is connected to an ethernet (or a wifi if you have one) and does not need a _modem_ connection.
	```bash
	sudo systemctl stop ModemManager 
	sudo systemctl disable ModemManager 
	```
5. _unattended-upgrades_ is a daemon to automatically update the system. I like when a computer works for me but in this specific case, we will avoid doing any automatic update. The reason is we want a stable entertainment system for all the family, which is available at any time. We don't want to have to do maintenance, just before launching the family movie, because an update didn't work:
	```bash
	sudo systemctl stop unattended-upgrades 
	sudo systemctl disable unattended-upgrades 
	```

	If you want to upgrade your Odroid XU4, you can still do it manually by running first an update of the packages' database:
	```bash
	sudo apt update
	```

	Then if you like the list of packages to be updated, run an upgrade:
	```bash
	sudo apt upgrade
	```

	Personally, I'm a big fan of a software called `synaptic` which is a GUI for apt-based systems like Ubutun and Debian. I recommend it.
