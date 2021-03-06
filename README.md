# How to install Kodi and transform your ODroid XU4 into a TV box and media center
**VERSION 2.0 - 2 January 2021**

This guide will explain how to make a TV box with your ODroid XU4. The first section is the only one you need to read in order to have a fully functional TV box using Kodi.
The next sections give extra configuration to set up an external USB drive, a MAME video game console and other topics.

One important feature is this new guide is that we don't use X11 or Wayland but Kodi runs directly on the Linux framebuffer using OpenGL acceleration thanks to the Mali 3d driver. Therefore the final product will be a very fast and light system. It is even possible to use it as a small home server for other purposes, even while playing videos. So all the family will be happy at the same time.

##### Table of contents
1. [Install Linux and Kodi on your Odroid XU4](#Install-Linux-and-Kodi-on-your-Odroid-XU4)<br/>
2. [Install an USB external drive on your Odroid XU4](#Install-an-USB-external-drive-on-your-Odroid-XU4)<br/>
3. [Install MAME to play arcade games from Kodi](#Install-MAME-to-play-arcade-games-from-Kodi)<br/>
4. [Install a joystick to play with MAME](#Install-a-joystick-to-play-with-MAME)<br/>

## Install Linux and Kodi on your Odroid XU4

All along this guide, I will assume you have access to a Linux box and all the command lines below are for Linux, either your other Linux machine or the Odroid XU-4 we're going to install now.
To install all of this, follow the steps:

1. The first step is ideally done on another Linux box. If you don't have one yet (_wait,what?_), better to migrate to Linux as soon as you can

	Download the Ubuntu image and its MD5 sum file:
    - https://dn.odroid.com/5422/ODROID-XU3/Ubuntu/ubuntu-20.04.1-5.4-minimal-odroid-xu4-20200812.img.xz.md5sum
    - https://dn.odroid.com/5422/ODROID-XU3/Ubuntu/ubuntu-20.04.1-5.4-minimal-odroid-xu4-20200812.img.xz

2. Check your image is valid with `md5sum` and compare the result with the content of the previously downloaded md5sum above

	```bash
    md5sum ubuntu-20.04.1-5.4-minimal-odroid-xu4-20200812.img.xz
	```

    and compare the result with the content of the file `ubuntu-20.04.1-5.4-minimal-odroid-xu4-20200812.img.xz.md5sum`

3. Uncompress the file first you've downloaded:

	```bash
	xz -d ubuntu-20.04.1-5.4-minimal-odroid-xu4-20200812.img.xz 
	```

4. Put an SD card with enough capacity in your Linux machine

5. Search for the device with `lsblk`

	```bash
	lsblk
	```

	**WARNING**

	Let's assume your SD card is on `/dev/sdX`. Be very careful to choose the correct device or you will risk to erase the content of another disk.

	**WARNING**

6. Write the image to the SD card

	```bash
	dd if=ubuntu-20.04.1-5.4-minimal-odroid-xu4-20200812.img.xz of=/dev/sdX bs=1M conv=fsync status=progress
	sync
	```

7. Umount your SD card either from your desktop environment or on the command line with. This step is not required but can be necessary:

	```bash
	sudo umount /dev/sdX
	```

8. Insert your SD card in your odroid XU4 and boot your machine

    Check the release notes pages to find the credentials:
    https://wiki.odroid.com/odroid-xu4/os_images/linux/ubuntu_5.4/minimal/20200812
    Usually, the login is `root` and the password is `odroid`. Obviously, you want to change them immediately after logging in.

	At this stage you have a fully functional ODroid XU4 machine. The next steps can either be done from your XU4 by connecting a keyboard to it or you can SSH from another machine to the XU4. I recommend using the Ethernet port at first. It is much simpler to configure than a wifi. But both wifi and Ethernet will work.

9. _Optional_: if you want to SSH to your XU4, you must first find the IP address of your XU4:

    ```bash
    ip addr
    ```
    The IP address is after the `inet` keyboard. The result of the command above gives a list of at least two entries. Usually the first one is called `lo` and is the _loopback_ network interface. It's a fake network interface which creates a network with one machine only. It is used if you want to test a networked program for which the client and the server applications are on the same machine.
    Usually, the second entry is _eth0_ and denotes the Ethernet port of your XU4. It's the one you want and the IP address you need is after the `inet` keyword. Use this address from your other machine to connect to your XU4

10. Connect as `root` as explained above, either directly or through a SSH connection to your XU4. Alternatively you can connect with the default `odroid` user and open a terminal. Then in the terminal do `sudo bash` to login as `root`.

    To connect from another machine with `ssh`:

    ```bash
    ssh root@192.168.xxx.zzz
    ```

    where `xxx` and `zzz` are replaced by the numbers you found doing `ip addr`.

11. Update the packages

	```bash
	apt update
	apt upgrade
	apt dist-upgrade
	```

	During the update, you might encounter the following situations (or not):
    - the first time you boot your machine, you may get an error regarding a lock file with dpkg (`/var/lib/dpkg/lock`), it means that one of the automated update procedure is still running which is very likely with a new install. Just let it run until the end and try the commands again.

	- if you get a message about boot.ini being replaced, just say 'OK'

	- if you get a question about `/etc/apt-fast.conf`, answer Y (for Yes)

    The last command (`autoremove`) is to leave the system in a clean state and use a bit less disk by removing unecessary packages.

12. The Mali graphical driver is installed by default and works well. However, the driver provided by ARM only implement OpenGL ES and not OpenGL. We will use `gl4es` as a wrapper between OpenGL and OpenGL ES and therefore have a hardware-accelerated OpenGL on the Mali GPU in XU4:

	```bash
	apt install git cmake xorg-dev gcc g++ build-essential
	git clone https://github.com/ptitSeb/gl4es.git
	cd gl4es
	```

	Then we compile `gl4es`:
	```bash
	mkdir build
	cd build
	cmake .. -DODROID=1
	make -j
	```

	It will compile and display its progress with a lot of messages. Toward the end, if the compilation has been successfull, you should see the following:
	```
	[100%] Built target GL
	```

	Now we backup the previous GL library and install the new one:

	```bash
    mv /usr/lib/arm-linux-gnueabihf/libGL.so.1.7.0 /usr/lib/arm-linux-gnueabihf/save.libGL.so.1.7.0
    rm /usr/lib/arm-linux-gnueabihf/libGL.so.1
    cd ../lib
    cp libGL.so.1 /usr/lib/arm-linux-gnueabihf/
    ln -s /usr/lib/arm-linux-gnueabihf/libGL.so.1 /usr/lib/arm-linux-gnueabihf/libGL.so
    ldconfig
	```
13. Install Kodi now and disable the main GUI environment:

	```bash
    apt install kodi-fbdev kodi-fbdev-bin kodi-fbdev-data libcec4 ffmpeg libavcodec-extra libavcodec-extra58 libavdevice58 libavfilter-extra libavformat58 libdlna0 libpostproc55 libswresample3 libswscale5 
	```

14. For security reason, kodi and its associated programs will be run by a user with limited privileges, with no password and automatic login. We will call the user `kodi`:

	```bash
	adduser --disabled-password --gecos "" kodi
	```

	The output will be:
	```
	Adding user `kodi' ...
	Adding new group `kodi' (1000) ...
	Adding new user `kodi' (1000) with group `kodi' ...
	Creating home directory `/home/kodi' ...
	Copying files from `/etc/skel' ...
	```

15. Assign some privileges to this user:

	```bash
	usermod -a -G cdrom,video,plugdev,users,dialout,dip,input,netdev,audio,pulse,pulse-access,games kodi
	```

16. Add the Kodi service to `systemd` (better to copy and paste the following rather than typing it):

	```bash
	cat << EOF | tee /etc/systemd/system/kodi.service
	[Unit]
	Description = Kodi Media Center

	After = systemd-user-sessions.service network.target sound.target mysql.service dbus.service polkit.service upower.service 
	Wants = mysql.service pulseaudio.service

	[Service]
	User = kodi
	Group = kodi
	Type = simple
	#PAMName = login # you might want to try this one, did not work on all systems
	ExecStart = /usr/bin/kodi-standalone
	Restart = on-abort
	RestartSec = 5

	[Install]
	WantedBy = multi-user.target
	EOF
	```

18. Enable Kodi at boot time:

	```bash
	systemctl enable kodi
	systemctl set-default multi-user.target
	```

19. Remove the blinking cursor on the Kodi screen. There is an annoying blinking cursor at the top-left corner. This will remove it:
    
    ```bash
    grep -v 'exit 0' /etc/rc.local > /tmp/rc.local
    echo 'echo 0 > /sys/class/graphics/fbcon/cursor_blink' >> /tmp/rc.local
    echo 'exit 0' >> /tmp/rc.local
    mv /tmp/rc.local /etc/rc.local
    chmod ugo+x /etc/rc.local
    ```

20. Disable the `unattended upgrades` service to avoid packages being installed in the background. It caused a few problems in the past and we want a stable Kodi system for all the family:

    ```bash
    systemctl stop unattended-upgrades.service
    systemctl disable unattended-upgrades.service
    ```

21. Add support for audio and the PulseAudio service

	```bash
    systemctl --user enable pulseaudio.service
    ```

    The Kodi service requires the PulseAudio service (cf. the `Wants` line above). PulseAudio comes by default in Ubuntu, so we just need to link to it.
    When Kodi starts, it will start the PulseAudio service just before and give Kodi a sound service on-demand.

22. Clean up and remove unused services to free up memory and CPU cycles:

    ModemManager is a daemon which controls mobile broadband (2G/3G/4G) devices and connections. The Odroid XU4 is connected to an ethernet (or a wifi if you have one) and does not need a _modem_ connection.

	```bash
	systemctl stop ModemManager 
	systemctl disable ModemManager 
	```
23. Set the clock to your own time zone

    You can synchronize the clock to a time server on the net and always have your Odroid set to the most accurate time. Moreover, you want to set the clock to your timezone.

    1. First we install an [NTP](https://en.wikipedia.org/wiki/Network_Time_Protocol) (Network Time Protocol) client which will connect to time server to get an accurate time
	```bash
	apt install chrony
	```

    2. Then we search for the time zone. It will be something like `Europe/Zurich` or `Pacific/Auckland`. To find yours do:
	```bash
	timedatectl list-timezones
	```
	Search and note your own time zone. Let's say you live near the North Pole in [Longyearbyen](https://en.wikipedia.org/wiki/Longyearbyen), you will find the time zone in the list given above as `Arctic/Longyearbyen`. Then tune your Odroid XU4 by doing:
	```bash
	timedatectl set-timezone Arctic/Longyearbyen
	```

24.	**You now have a fully functional Kodi system**. If you want to reboot, your Odroid XU4 will directly start on Kodi. I you want to add more features, like [MAME](https://www.mamedev.org/) ([Mame on Wikipedia](https://en.wikipedia.org/wiki/MAME)), an external USB drive, a joystick, a firewall, just keep reading.

In the next sections, I'll show you how to:
- install an external USB drive and have it automounted (for Kodi or Mame for example),
- install and configure Mame and use it from Kodi
- install and configure a joystick for Mame
- remove software and services which are not necessary and use memory and CPU cycle for nothing

Everything can be done either by connecting to your ODroid XU4 with `ssh` or on the screen directly (you'll need a keyboard). For the second option, after booting on the Kodi screen, you need to exit Kodi first to get back onto the terminal. After doing what needs to be done (below), just reboot your machine to check everything is fine.

## Install an USB external drive on your Odroid XU4


1. Switch to a terminal or login with `ssh`.

2. Install `usbmount` to automount USB drives:

	```bash
	apt install usbmount debhelper build-essential fakeroot
	```

	There is a bug in the version 0.0.22 of `usbmount` as provided by the stock Ubuntu Linux when connecting 2 USB drives at the same time. The bug has been fixed with `usbmount 0.0.24` which is not yet on the Ubuntu repository. You can upgrade it manually, if you like:

	If your second drive (or even the first) doesn't mount properly when you plug it in, you can try to upgrade `usbmount` as follows:

	```bash
	git clone https://github.com/rbrito/usbmount.git
	cd usbmount
	dpkg-buildpackage -us -uc -b
	cd ..
	dpkg -i usbmount_0.0.24_all.deb
	```

3. By default, usbmount will mount the external USB drive with the `sync` option. `sync` means that all write access to the disk will be immediately flushed to the disk. And it can dramatically slow down all the disk operations. 

	```bash
	sed -ie 's/^MOUNTOPTIONS=.*/MOUNTOPTIONS="noexec,nodev,noatime,nodiratime"/' /etc/usbmount/usbmount.conf
	```
	I assume it is safe to use the `async` option here because you're not supposed to unplug this disk at any time. When I've done that, the write speed of my external hard drive has been multiplied by 10!

## Install MAME to play arcade games from Kodi

This section is long and will require external resources if you want to have all the bells and whistles of a full-featured MAME installation. From time to time, we will refer to external guides too.

1. Install MAME

	```bash
	sudo apt install mame mame-data mame-doc mame-extra mame-tools libsdl2-gfx-1.0-0 libsdl2-image-2.0-0 libsdl2-mixer-2.0-0 libsdl2-net-2.0-0 libsdl2-net-2.0-0 python-pil
	```

	Create a new config file with the correct paths, using OpenGL ES 2, the ALSA audio driver, tuning the speed and using all the cores:
	```bash
	cat << EOF | sudo tee /etc/mame/mame.ini
    inipath                  $HOME/.mame;/etc/mame
	rompath                 /home/kodi/AML-ROMs/
	samplepath              /home/kodi/AML-assets/samples/
	ctrlrpath               /home/kodi/AML-assets/ctrlr
	hashpath                /usr/share/games/mame/hash

	cfg_directory           $HOME/.mame/cfg
	nvram_directory         $HOME/.mame/nvram
	memcard_directory       $HOME/.mame/memcard
	input_directory         $HOME/.mame/inp
	state_directory         $HOME/.mame/sta
	snapshot_directory      $HOME/.mame/snap
	diff_directory          $HOME/.mame/diff
	comment_directory       $HOME/.mame/comments

	video                   auto
	render                  opengles2
	audiodriver             alsa
	samplerate              32000
	numprocessors           8
	mouse                   1
	uimodekey               INSERT
	autoframeskip           1
	EOF
	sudo chmod ugo+r /etc/mame/mame.ini
	```

	Then we need to remove the `gl4es` startup message to make MAME happy. Long story short: a Kodi plugin will extract the games' database `MAME.xml` to the standard output by running `mame`. When `mame` starts, `gl4es` is initialized and displays a nice message, like the one above. But the Kodi plugins capture the standard output which is supposed to be an XML file, except that we have this welcome message on top of it from `gl4es`. So when Kodi tries to read the XML file (in fact a Python library tries too), it fails:
	```bash
	cat << EOF | sudo tee /usr/local/bin/mame
	#!/bin/sh
	export LIBGL_SILENTSTUB=1
	export LIBGL_NOBANNER=1
	/usr/games/mame "$@"
	EOF
	sudo chmod ugo+x /usr/local/bin/mame
	```

	When configuring the AML add-on in Kodi, we will use this new mame command we've just created

2. Install and configure Advanced Mame Launcher plugin from Kodi

	We're going to follow the official guide of a Kodi plugin called [Advanced Mame Launcher (AML)](https://forum.kodi.tv/showthread.php?tid=304186) and adapt a few steps to follow our setup. Here, I assume that we have an external USB drive connected to the Odroid XU4. It will be helpful to store data for MAME.

	1. Assuming you're on a text terminal (see above on how to switch to a text terminal from Kodi), the first step is to create directories to store MAME data. The external USB drive is mounted to `/media/usb0` and by default to `/media/usb` too. We will assume it's the case from now on.

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
	
	2. Then we need to fill in the directories with the latest data for Mame as described in https://forum.kodi.tv/showthread.php?tid=304186. We use the following script to download and install everything automatically:

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

3. Modify `mame.ini` to reflect the new directory structure:

	```bash
	sudo sed -i 's#^rompath \+.*$#rompath /home/kodi/AML-ROMs/#' /etc/mame/mame.ini
	sudo sed -i 's#^samplepath \+.*$#samplepath /home/kodi/AML-assets/samples/#' /etc/mame/mame.ini
	```

4. Add assets
	1. Go back to Kodi: type 'Ctrl+D' on your terminal, then 'Atl+F7' to go back to Kodi.

	2.  In https://forum.kodi.tv/showthread.php?tid=304186, you can follow the paragraph called _Setting up MAME assets and Software List assets_ to add more resources and assets. This is the way to get extra pictures, logos, etc...
		You can read the page here: http://forum.pleasuredome.org.uk/index.php?showtopic=30715 about the MAME Extra packages to understand all the type of resources you can find on the Net for Mame.

	3. On the above mentioned page, you can find links to Mame resources on the Internet. They come in huge packages you simply have to move to `/media/usb/mame` as we created before. Let me give you some examples about those packages, which can be useful in the process:
		- if you find a package called `MAME 0.xxx EXTRAs` where `xxx` is a Mame version number, go into the directory you have downloaded. In this directory, you will find more directories. Move their content to the corresponding directory in `/media/usb/mame/AML-assets/<some directory>`
		- Another similar package might be called something like `MAME 0.xxx Multimedia`. You should process it the same way
		- but you will find `.zip` files too. You can simply unzip them and move them to the final directory as above. Some directories don't have the same name, so keep the names we created above. In this case, transfer the content of the directory instead of the directory itself.
		- if you find packages with `bios-devices` in their name you don't need them if you already downloaded the very big packages with `ROMs` in their name. They are just subset for different situation. You can read more about all those packages [on this page](http://forum.pleasuredome.org.uk/index.php?showtopic=34705). These packages are made when you want a minimal version of the ROMs for example if you're running short on disk.
		- There are packages called Rollbacks Roms too ([information here](http://forum.pleasuredome.org.uk/index.php?showtopic=30284#entry260352)). You only need them if you have a ROM manager for Mame which can deal with multiple version. You won't need them in this tutorial.
		- There are the Software List ROMs and CHDs. Obviously, their content will go into the `/media/usb/mame/AML-SL-ROMS` and `/media/usb/mame/AML-SL-CHDs` directories we've created before. `SL` stands for `Software List` of course.


	4. Finally, you might have messed up a bit with users' permission when downloading and moving files. So you want to make things well by assignign the `kodi` user to the `mame` directory:
	```bash
	sudo chown -R kodi.kodi /media/usb/mame
	```

5. Install and configure the AML plugin. You will find it in Program Adds-on. It's called _Advanced Mame Launcher_.
	1. in Kodi, go to **Settings**, **Addon settings**, **Install from repository**. In **Program add-ons**, look for **Advanced Mame Launcher** and install it. Follow the picture in the order:
	![amlconf001](/images/amlconf001.png)

	2. After it's installed, right-click on the add-ons logo and go to _Settings_:
	![amlconf002](/images/amlconf002.png)

	3. Change the paths to the executable and the data directories as shown on the picture:
	![amlconf003](/images/amlconf003.png)

	4. Go back to Kodi's initial screen and look for the AML plugin, in general **Add-ons**, **Program Add-ons**, **Advanced Mame Launcher**.

	5. Select any row, open the context menu with a right-click and select **Setup plugin**.

	6. And run a full setup and configuration of the plugin by choosing the _All in one step_ options in the context menu of the plugin:
	![amlconf004](/images/amlconf004.png)
	This step can take several minutes to an hour. You will see a lots of progress bars. If you did everything well before, it should work without error messages. The plugin is now configured and ready.

<br/>

**And Mame is ready!**

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
	- another option (since Kodi 17) is to setup your joystick directly from Kodi. Read the [tutorial here](https://kodi.wiki/view/HOW-TO:Configure_controllers).

	As there are many models of joystick, I won't cover all the possible configurations but please contribute and I'll add your solution to this guide.

	- If you want to play with the joystick configuration and assign various command to each button or change the directions, you need to provide a configuration file. Let's call it `myjoyremap.cfg`. In my case, I use the following file, but yours might differ a lot depending on what you want to achieve and your joystick's model.
	```xml
	<?xml version="1.0"?>
	<mameconfig version="10">
	    <system name="default">
	        <input> 
		    <port type="P1_JOYSTICK_UP"> <newseq type="standard"> JOYCODE_1_XAXIS_RIGHT_SWITCH </newseq> </port>
	            <port type="P1_JOYSTICK_DOWN"> <newseq type="standard"> JOYCODE_1_XAXIS_LEFT_SWITCH </newseq> </port>
	            <port type="P1_JOYSTICK_LEFT"> <newseq type="standard"> JOYCODE_1_YAXIS_UP_SWITCH </newseq> </port>
	            <port type="P1_JOYSTICK_RIGHT"> <newseq type="standard"> JOYCODE_1_YAXIS_DOWN_SWITCH </newseq> </port>
	
	            <port type="P1_BUTTON1"> <newseq type="standard"> JOYCODE_1_BUTTON1 </newseq> </port>
	            <port type="P1_BUTTON2"> <newseq type="standard"> JOYCODE_1_BUTTON3 </newseq> </port>
	            <port type="P1_BUTTON3"> <newseq type="standard"> JOYCODE_1_BUTTON5 </newseq> </port>
	            <port type="P1_BUTTON4"> <newseq type="standard"> JOYCODE_1_BUTTON7 </newseq> </port>
	
	            <port type="P1_BUTTON5"> <newseq type="standard"> JOYCODE_1_BUTTON2 </newseq> </port>
	            <port type="P1_BUTTON6"> <newseq type="standard"> JOYCODE_1_BUTTON4 </newseq> </port>
	            <port type="P1_BUTTON7"> <newseq type="standard"> JOYCODE_1_BUTTON6 </newseq> </port>
	
	            <port type="P1_BUTTON8"> <newseq type="standard"> NONE </newseq> </port>
	            <port type="P1_BUTTON9"> <newseq type="standard"> NONE </newseq> </port>
	            <port type="P1_BUTTON10"> <newseq type="standard"> NONE </newseq> </port>
	            <port type="P1_BUTTON11"> <newseq type="standard"> NONE </newseq> </port>
	            <port type="P1_BUTTON12"> <newseq type="standard"> NONE </newseq> </port>
	
	            <port type="P1_START"> <newseq type="standard"> JOYCODE_1_BUTTON9 </newseq> </port>
	            <port type="P1_SELECT"> <newseq type="standard"> JOYCODE_1_BUTTON10 </newseq> </port>
	            <port type="COIN1"> <newseq type="standard"> JOYCODE_1_BUTTON8 </newseq> </port>
	            <port type="START1"> <newseq type="standard"> KEYCODE_1 OR JOYCODE_1_BUTTON9 </newseq> </port>
	
	            <port type="P1_PEDAL"> <newseq type="standard"> NONE </newseq> <newseq type="increment"> NONE </newseq>
	            </port> <port type="P1_PEDAL2"> <newseq type="standard"> NONE </newseq> <newseq type="increment"> NONE </newseq> </port>
	            <port type="P1_PEDAL3"> <newseq type="increment"> NONE </newseq> </port>
	            <port type="P1_PADDLE"> <newseq type="standard"> NONE </newseq> <newseq type="increment"> NONE </newseq> <newseq type="decrement"> NONE </newseq> </port>
	            <port type="P1_PADDLE_V"> <newseq type="standard"> NONE </newseq> <newseq type="increment"> NONE </newseq> <newseq type="decrement"> NONE </newseq> </port>
	            <port type="P1_POSITIONAL"> <newseq type="standard"> NONE </newseq> <newseq type="increment"> NONE </newseq> <newseq type="decrement"> NONE </newseq> </port>
	            <port type="P1_POSITIONAL_V"> <newseq type="standard"> NONE </newseq> <newseq type="increment"> NONE </newseq> <newseq type="decrement"> NONE </newseq> </port>
	            <port type="P1_DIAL"> <newseq type="standard"> NONE </newseq> <newseq type="increment"> NONE </newseq> <newseq type="decrement"> NONE </newseq> </port>
	            <port type="P1_DIAL_V"> <newseq type="standard"> NONE </newseq> <newseq type="increment"> NONE </newseq> <newseq type="decrement"> NONE </newseq> </port>
	            <port type="P1_TRACKBALL_X"> <newseq type="standard"> NONE </newseq> <newseq type="increment"> NONE </newseq> <newseq type="decrement"> NONE </newseq> </port>
	            <port type="P1_TRACKBALL_Y"> <newseq type="standard"> NONE </newseq> <newseq type="increment"> NONE </newseq> <newseq type="decrement"> NONE </newseq> </port>
	            <port type="P1_AD_STICK_X"> <newseq type="standard"> NONE </newseq> <newseq type="increment"> NONE </newseq> <newseq type="decrement"> NONE </newseq> </port>
	            <port type="P1_AD_STICK_Y"> <newseq type="standard"> NONE </newseq> <newseq type="increment"> NONE </newseq> <newseq type="decrement"> NONE </newseq> </port>
	            <port type="P1_AD_STICK_Z"> <newseq type="standard"> NONE </newseq> <newseq type="increment"> NONE </newseq> <newseq type="decrement"> NONE </newseq> </port>
	            <port type="P1_LIGHTGUN_X"> <newseq type="standard"> NONE </newseq> <newseq type="increment"> NONE </newseq> <newseq type="decrement"> NONE </newseq> </port>
	            <port type="P1_LIGHTGUN_Y"> <newseq type="standard"> NONE </newseq> <newseq type="increment"> NONE </newseq> <newseq type="decrement"> NONE </newseq> </port>
	
	
	
		    <port type="P2_JOYSTICK_UP"> <newseq type="standard"> JOYCODE_2_XAXIS_RIGHT_SWITCH </newseq> </port>
	            <port type="P2_JOYSTICK_DOWN"> <newseq type="standard"> JOYCODE_2_XAXIS_LEFT_SWITCH </newseq> </port>
	            <port type="P2_JOYSTICK_LEFT"> <newseq type="standard"> JOYCODE_2_YAXIS_UP_SWITCH </newseq> </port>
	            <port type="P2_JOYSTICK_RIGHT"> <newseq type="standard"> JOYCODE_2_YAXIS_DOWN_SWITCH </newseq> </port>
	
	            <port type="P2_BUTTON1"> <newseq type="standard"> JOYCODE_2_BUTTON1 </newseq> </port>
	            <port type="P2_BUTTON2"> <newseq type="standard"> JOYCODE_2_BUTTON3 </newseq> </port>
	            <port type="P2_BUTTON3"> <newseq type="standard"> JOYCODE_2_BUTTON5 </newseq> </port>
	            <port type="P2_BUTTON4"> <newseq type="standard"> JOYCODE_2_BUTTON7 </newseq> </port>
	
	            <port type="P2_BUTTON5"> <newseq type="standard"> JOYCODE_2_BUTTON2 </newseq> </port>
	            <port type="P2_BUTTON6"> <newseq type="standard"> JOYCODE_2_BUTTON4 </newseq> </port>
	            <port type="P2_BUTTON7"> <newseq type="standard"> JOYCODE_2_BUTTON6 </newseq> </port>
	
	            <port type="P2_BUTTON8"> <newseq type="standard"> NONE </newseq> </port>
	            <port type="P2_BUTTON9"> <newseq type="standard"> NONE </newseq> </port>
	            <port type="P2_BUTTON10"> <newseq type="standard"> NONE </newseq> </port>
	            <port type="P2_BUTTON11"> <newseq type="standard"> NONE </newseq> </port>
	            <port type="P2_BUTTON12"> <newseq type="standard"> NONE </newseq> </port>
	
	            <port type="P2_START"> <newseq type="standard"> JOYCODE_2_BUTTON9 </newseq> </port>
	            <port type="P2_SELECT"> <newseq type="standard"> JOYCODE_2_BUTTON10 </newseq> </port>
	            <port type="COIN2"> <newseq type="standard"> JOYCODE_2_BUTTON8 </newseq> </port>
	            <port type="START2"> <newseq type="standard"> KEYCODE_2 OR JOYCODE_2_BUTTON9 </newseq> </port>
	
	            <port type="P2_PEDAL"> <newseq type="standard"> NONE </newseq> <newseq type="increment"> NONE </newseq>
	            </port> <port type="P2_PEDAL2"> <newseq type="standard"> NONE </newseq> <newseq type="increment"> NONE </newseq> </port>
	            <port type="P2_PEDAL3"> <newseq type="increment"> NONE </newseq> </port>
	            <port type="P2_PADDLE"> <newseq type="standard"> NONE </newseq> <newseq type="increment"> NONE </newseq> <newseq type="decrement"> NONE </newseq> </port>
	            <port type="P2_PADDLE_V"> <newseq type="standard"> NONE </newseq> <newseq type="increment"> NONE </newseq> <newseq type="decrement"> NONE </newseq> </port>
	            <port type="P2_POSITIONAL"> <newseq type="standard"> NONE </newseq> <newseq type="increment"> NONE </newseq> <newseq type="decrement"> NONE </newseq> </port>
	            <port type="P2_POSITIONAL_V"> <newseq type="standard"> NONE </newseq> <newseq type="increment"> NONE </newseq> <newseq type="decrement"> NONE </newseq> </port>
	            <port type="P2_DIAL"> <newseq type="standard"> NONE </newseq> <newseq type="increment"> NONE </newseq> <newseq type="decrement"> NONE </newseq> </port>
	            <port type="P2_DIAL_V"> <newseq type="standard"> NONE </newseq> <newseq type="increment"> NONE </newseq> <newseq type="decrement"> NONE </newseq> </port>
	            <port type="P2_TRACKBALL_X"> <newseq type="standard"> NONE </newseq> <newseq type="increment"> NONE </newseq> <newseq type="decrement"> NONE </newseq> </port>
	            <port type="P2_TRACKBALL_Y"> <newseq type="standard"> NONE </newseq> <newseq type="increment"> NONE </newseq> <newseq type="decrement"> NONE </newseq> </port>
	            <port type="P2_AD_STICK_X"> <newseq type="standard"> NONE </newseq> <newseq type="increment"> NONE </newseq> <newseq type="decrement"> NONE </newseq> </port>
	            <port type="P2_AD_STICK_Y"> <newseq type="standard"> NONE </newseq> <newseq type="increment"> NONE </newseq> <newseq type="decrement"> NONE </newseq> </port>
	            <port type="P2_AD_STICK_Z"> <newseq type="standard"> NONE </newseq> <newseq type="increment"> NONE </newseq> <newseq type="decrement"> NONE </newseq> </port>
	            <port type="P2_LIGHTGUN_X"> <newseq type="standard"> NONE </newseq> <newseq type="increment"> NONE </newseq> <newseq type="decrement"> NONE </newseq> </port>
	            <port type="P2_LIGHTGUN_Y"> <newseq type="standard"> NONE </newseq> <newseq type="increment"> NONE </newseq> <newseq type="decrement"> NONE </newseq> </port>
	
	        </input>
	    </system>
	</mameconfig>
	```

	Then you need to copy it to `/media/usb/AML-assets/ctrlr/`.

	Then you can edit this file to adjust it to you own joystick. My joystick (which I built myself, hence the _little_ problem :-) ) has a 90 degrees misconfiguration. So UP is right, RIGHT is down, etc... In the XML file, you have two parts. one for the _Player 1_ joystick and another part for the _Player 2_. Each line with `type="P1_JOYSTICK_UP"` (etc...) is the direction as understood by `mame`. Then, the real configuration comes after as `JOYCODE_1_XAXIS_RIGHT_SWITCH`. Therefore, this line means that when my joystick sends a code for `RIGHT`, `mame` will interpret it as `UP`. Below this, I configured the buttons 1 to 8 and the `START` and `SELECT` buttons. Then I do the same for the Player's 2 joystick.

	Finally, you can add it to the `mame.ini` configuration file by doing
	```bash
	echo "ctrlr myjoyremap" | sudo tee -a /etc/mame/mame.ini
	```

---
### Support my work by making a small donation
<img alt="" border="0" src="https://www.paypal.com/en_GB/i/scr/pixel.gif" width="1" height="1" />

- with **Paypal**:
	<a href="https://paypal.me/DavidBellot?locale.x=en_GB">
		<img alt="Donate with PayPal button" style="border-width:0" src="https://www.paypalobjects.com/en_US/GB/i/btn/btn_donateCC_LG.gif"  />	
	</a>

- with Bitcoin at the address: **18kuqN4vsy4ALNrg39dCkQnEoPPEbqagsH** or by using the following QR code:
	![BTC donations](/images/btc.png)

---
<a rel="license" href="http://creativecommons.org/licenses/by-sa/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-sa/4.0/88x31.png" /></a><br /><span xmlns:dct="http://purl.org/dc/terms/" property="dct:title">odroid-xu4-setup</span> by <a xmlns:cc="http://creativecommons.org/ns#" href="https://github.com/yimyom/odroid-xu4-setup" property="cc:attributionName" rel="cc:attributionURL">David Bellot</a> is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by-sa/4.0/">Creative Commons Attribution-ShareAlike 4.0 International License</a>.<br />Based on a work at <a xmlns:dct="http://purl.org/dc/terms/" href="https://github.com/yimyom/odroid-xu4-setup" rel="dct:source">https://github.com/yimyom/odroid-xu4-setup</a>.

###### All the images in this document are licensed under [CC-BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/)

---
David Bellot
Sydney, Australia
2019-2021
