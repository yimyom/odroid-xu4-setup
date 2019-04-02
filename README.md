**WARNING: This is a work in progress as of 3.4.2019. Don't use it yet, until the version number below reaches 1.0**

# odroid-xu4-setup
**VERSION 0.1**

How to set up an ODroid XU4 with Kodi, Mame and an external USB drive

I describe all the steps to install Ubuntu Linux on an ODroid XU4 and use it as a media and game center using Kodi and Mame.
Installing Mame is optional as well as configuring the USB drive, the joystick, etc... You can stop at the Kodi level or go further and have a fully functional video, music and game machine

To install all of this, follow the steps:

1. install from 
https://wiki.odroid.com/odroid-xu4/os_images/linux/ubuntu_4.14/20171213

2. md5sum ubuntu-16.04.3-4.14-minimal-odroid-xu4-20171213.img.xz

...and check it corresponds to http://de.eu.odroid.in/ubuntu_16.04lts/ubuntu-16.04.3-4.14-minimal-odroid-xu4-20171213.img.xz.md5sum

3. xz -d ubuntu-16.04.3-4.14-minimal-odroid-xu4-20171213.img.xz

4. plugin sd card in usb port
5. search for the device with lsblk
6. write the image
...dd if=ubuntu-16.04.3-4.14-minimal-odroid-xu4-20171213.img of=/dev/sdXXX bs=1M conv=fsync
...where XXX is your device
...sync
7. umount your sdcard (from the desktop) or with a umount /dev/sdXXX
8. insert your sdcard in your odroid XU4 and boot.
...Then login on the desktop.
...The next stage is to remove direct desktop login and replace it with kodi

9. login on the desktop with the default user (odroid)
10. open a terminal (search in the menu on the top-left of the screen)
11. update the packages
...sudo apt update
...sudo apt upgrade
...sudo apt dist-upgrade

...At each time, it might ask your for the odroid password. It's "odroid".
...You might want to change it for safety reasons
...In the terminal, you can type
...passwd
...then answer the questions (old and new password)
...Next time, use this password

...During the update, the first time you boot your machine, you may get an error regarding a lock file with dpkg (/var/lib/dpkg/lock), it means that one of the automated update procedure is still running which is very likely with a new install. Just let it run until the end and try the commands again from step 11.

...During your update, if you get a message about boot.ini being replaced, just say 'OK'

...During the update, if you get a question about /etc/apt-fast.conf, answer Y (for Yes)

13. login again and open a terminal (see step 10)
14. Kodi is installed by default
15. Mali driver is installed by default and works well. Go in a terminal and run
...glmark2-es2 and you should see a demo (a horse). FPS should be around 300 frames/second
...run glmark2 and you should see the same demo. FPS should be around 30 frames/second.
...The difference is due to the fact that the Mali driver only supports OpenGL ES and not plain OpenGL
...There is a solution for that

16. Install gl4es

In a terminal again:
...cd
...sudo apt install git cmake xorg-dev

...get the source code for a library wrapping OpenGL to OpenGL-ES
...git clone https://github.com/ptitSeb/gl4es.git
...cd gl4es

...Let's compile it now
...mkdir build
...cd build
...cmake .. -DODROID=1
...make

...It will compile and display its progress.
...If you see this at the end:
...[ 96%] Building C object src/CMakeFiles/GL.dir/glx/utils.c.o
...[ 98%] Linking C shared library ../lib/libGL.so.1
...[100%] Built target 

...Then it's a success and you can proceed to the next step

...Save the mesa version of OpenGL (the one installed by default)

...sudo mv /usr/lib/arm-linux-gnueabihf/libGL.so.1 /usr/lib/arm-linux-gnueabihf/save.libGL.so.1
...sudo mv /usr/lib/arm-linux-gnueabihf/libGL.so.1.0.0 /usr/lib/arm-linux-gnueabihf/save.libGL.so.1.0.0

...and copy your new version
...sudo cp lib/libGL.so.1 /usr/lib/arm-linux-gnueabihf/
...sudo ldconfig

...Now check that everything is fine by running
...glxinfo|head -6

...If you see 
...LIBGL: Initialising gl4es
...LIBGL: v1.1.1 built on Mar 28 2019 06:22:26
...LIBGL: Using GLES 2.0 backend
...LIBGL: loaded: libGLESv2.so
...LIBGL: loaded: libEGL.so
...LIBGL: Using GLES 2.0 backend

...(of course the date on the second line will be different for you)

...if you run
...glmark2

...this time you should see FPS around 300

...and now your ODroid-XU4 has full opengl/opengl-es support

19. For security reason, kodi and its associated programs will be run by a user with limited privileges, with no password and automatic login. We will call the user 'kodi'
...adduser --disabled-password --gecos "" kodi

...Adding user `kodi' ...
...Adding new group `kodi' (1000) ...
...Adding new user `kodi' (1000) with group `kodi' ...
...Creating home directory `/home/kodi' ...
...Copying files from `/etc/skel' ...

20. assign some privileges to this user
...usermod -a -G cdrom,video,plugdev,users,dialout,dip,input,netdev,audio,pulse kodi

21. Add options to allow Kodi to start it owns X server
...sed -ie 's/allowed_users=console/allowed_users=anybody/g' /etc/X11/Xwrapper.config
...sed -ie "\$aneeds_root_rights = yes" /etc/X11/Xwrapper.config

22. Add the kodi service to systemd
...cat > /etc/systemd/system/kodi.service <<EOF
...[Unit]
...Description = Kodi Media Center

...After = systemd-user-sessions.service network.target sound.target mysql.service
...Wants = mysql.service

...[Service]
...User = kodi
...Group = kodi
...Type = simple
...#PAMName = login # you might want to try this one, did not work on all systems
...ExecStart = /usr/bin/xinit /usr/bin/dbus-launch --exit-with-session /usr/bin/kodi-standalone -- :0 -nolisten tcp vt7
...Restart = on-abort
...RestartSec = 5

...[Install]
...WantedBy = multi-user.target
...EOF

23. Enable Kodi at boot
...sudo systemctl enable kodi
...sudo systemctl set-default multi-user.target

24. Install usbmount to add external drives automatically
...sudo apt install usbmount
...This is not mandatory but very helpful

25. From here you have a fully functional Kodi box which you can use directly
If you reboot with
...sudo reboot
you will have land on Kodi directly. If you want to install more features, stay on the terminal

...We are going on to the optional features now:
...- USB drive automount to store whatever you need to store for Kodi, Mame, etc...
...- Install and configure Mame from Kodi
...- Install and configure Joystick for Kodi
...- Remove some features to save memory and CPU cycle (a little bit)
...- firewall rules to set up the web connection for Kodi and use one of the many Android/iOS apps to control Kodi
...- create a swapfile

26. Install USB drive automount
...sudo apt install usbmount
...I found a bug in the version 0.0.22 of usbmount when connecting 2 drives. The bug has been fixed with usbmount 0.0.24.
...If your second drive (or even the first) doesn't mount properly when you plug it in, you can try to upgrade usbmount as follows:
...git clone https://github.com/rbrito/usbmount.git
...cd usbmount
...sudo apt-get update && sudo apt-get install -y debhelper build-essential
...dpkg-buildpackage -us -uc -b
...cd ..
...sudo dpkg -i usbmount_0.0.24_all.deb

27. Install MAME
...sudo apt install mame mame-data mame-doc mame-extra mame-tools

...Change mame driver to opengles
...(If someone has a solution to make it work with OpenGL and gl4es, please contact me)
...sudo sed -i 's/opengl/opengles/' /etc/mame/mame.ini

28. Install Advanced Mame Launcher plugin from Kodi
...It's best to follow the guide on https://forum.kodi.tv/showthread.php?tid=304186
...What I did was to plug-in a USB drive to my machine to store all the (very big) data required by Mame. You will need hundred of Gb if not more and it's more than the SD card in my Odroid XU4 can hold.

...If you already rebooted your Odroid XU4, there are 2 ways to access to a terminal now
... 1. Ctrl+Alt+F1 will switch to a Linux text terminal. Then you can log in with your odroid user
... 2. From another machine you can ssh to your Odroid XU4. If you don't know what it means, better to learn as ssh is an incredibly powerful and useful tool to know

What I did to make AML works is as follows: first of all, I login onto my odroid user (as described above)
Then, I will assume for the following that I have a drive connected to the USB port and this drive is identified by /media/usb

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

(the last exit brings you back to your odroid user)

This is the structure for running AML in Kodi. You will have to fill in those directories with data as described in https://forum.kodi.tv/showthread.php?tid=304186

Next we need to modify mame.ini to reflect this directories' structure:

sudo sed -i 's#^rompath \+.*$#rompath /home/kodi/AML-ROMs/#' /etc/mame/mame.ini
sudo sed -i 's#^samplepath \+.*$#samplepath /home/kodi/AML-assets/samples/#' /etc/mame/mame.ini

...You can edit the file instead or re-use the same 'sed' kind of lines to keep modifying mame.ini if you add more resources. You can download various .ini files with information about the games in Mame. Read the same installation guide. Once you've put everything you want, you can install the Kodi plug-in called Advanced Mame Launcher (AML).
... Again, from https://forum.kodi.tv/showthread.php?tid=304186, you can follow the paragraph called Setting up MAME assets and Software List assets to add more resources.
... Then follow the installation paragraph called Setting up Advanced MAME Launcher (Easy mode)
... And that's it. Mame is ready from Kodi

29. Install joystick support
sudo apt install joystick jstest-gtk

30. Calibrate the joystick
As most of Mame games will require a simple joystick with buttons, calibration will be very simple
You can use jstest-gtk but as we already install Kodi, we will do the calibration from the command line only. With click joystick, the only calibration is to associate buttons (inside the joystick for the directions and fire/select buttons), with their respective direction or function. The calibration will be available system-wide and therefore will be used by mame


31. Create a swap file to add more virtual memory
In order to _extend_ the memory of the Odroid XU4, we want to add a swap file. However, it's not a good idea to use the SD-card. It's faster but swap file can be written over and over a lot of time, thus reducing the life of the card. So we will use the external USB drive, connected as described above (where we assume the drive is always connected and mounted at /media/usb0):

...sudo fallocate -l 2G /media/usb0/swapfile
...sudo chmod 600 /media/usb0/swapfile
...sudo mkswap /media/usb0/swapfile
...sudo swapon /media/usb0/swapfile
...sudo sed -ie "\$a/media/usb0/swapfile swap swap defaults 0 0" /etc/fstab
