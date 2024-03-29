---
layout: post
author: jack
category: linux
---

I have been reusing my old laptop for storage purposes, which runs Alpine OS on it. Nothing professional but just for fun. It is a 6th generation i3 processor 6006 with dual core processor 2.0 GHz, which is no longer supported by intel. For some reason, high end OS doesn’t work correctly. If you have already read my old Alpine blog, you might have understood the situation with the installations of various OS. In this blog post, I will try to give an overview of all steps starting from installation to setting up the static IP address along with SSH logins and sudoers file

<b>Installation –</b>
Installing Alpine OS is very straight forward. The wiki from Alpine did provide an understanding of each step. Alpine doesn’t have a graphical window to install. Once you boot into the Alpine OS boot device, you should be welcomed with a shell and a message asking you to run setup-alpine to install the OS. I plugged in my ethernet cable to this machine so that I don’t have to deal with the networking prerequisites. If you do not have access to a wired connection, please refer to the networking section in the Alpine wiki.

Basically, the installation asks below things (not in the same order)-
<ol>
    <li>Keyboard layout – usually us-intl</li>
    <li>Host name</li>
    <li>Root password</li>
    <li>Network configuration</li>
    <ol>
        <li>In case of a wired connection, the interface should be detected and you will be asked for additional settings for every detected interface. For eg, in the case of wired connection, IP address configuration options are provided (DHCP or Static IP). We will be looking at the static IP address configuration in the end of this blog so, for now, we can leave the auto address allocation by DHCP.</li>
        <li>DHCP is something interesting to understand in general because this gives a sneak-peak into the networking from the routers perspective.</li>
    </ol>
    <li>Timezone</li>
    <li>Alpine package mirrors – which can be set by allowing the installer to take care of finding the right package for your location</li>
    <li>Partitions</li>
    <li>User and password creation</li>
</ol>

Once the installation is done successfully, you need to reboot the machine after unplugging the boot device

When you login back, go to /etc/apk/repositories and uncomment all the mirrors that are present so that we can get access to wide range of packages. Once done, install updates and upgrade your OS using apk update && apk upgrade.
<br><br>
<b>Networking (Static IP address) –</b>
Usually IP address allocation for a machine on a network is done by DHCP. But we can modify and assign a static IP address to take a few advantages – for eg, to get a static IP address for the SSH login. To do that, go to /etc/network/interfaces and edit the ethernet interface from dhcp configuration to static. In the end, the changes might look like below –

<div style="background-color:#c9c9c9">
    <p>iface eth0 inet static</p>
        <p>address 192.168.1.150/24</p>
        <p>gateway 192.168.1.1</p>
</div>




Try to adjust these numbers according to your network configuration where the address part is the static IP address you are planning to assign to the machine. Once done, restart the networking using /etc/init.d/networking restart and try running ping 8.8.8.8 to check if you are connected to the internet or not.
<br><br>
<b>SSH - </b>
Let’s setup the SSH keys so that this machine can be accessed from a different machine using SSH keys. Of course, password based access is always the default mode but not secure. So, we will be disabling the password based SSH logins and use pub-private keys to establish a connection. First generate a pub key on your machine from which you are planning to access this alpine machine, using ssh-keygen -t rsa . Now copy the content present inside the ~/.ssh/id_rsa.pub and paste it in the location ~/.ssh/authorized_keys on the alpine machine. For the first time, the folder and file might not present so create one before you paste public key inside the file. Now configure the SSH connection by editing the file /etc/ssh/sshd_config . By default, SSH uses port 22. To improve security, change the default port number. Next, disable root login and passwordbased login. It should be easy to find the keywords that address these functions. Once the settings are in place, restart the SSH daemon using /etc/init.d/ssh restart.

Try logging in from that remote machine to this Alpine machine using SSH. This time it should not be asking for password and should be using the public-private key method to establish the connection.
<br><br>
<b>Other settings –</b>
Apart from these above settings, you can also create a sudoers file and add yourself to that list so that you run root commands using sudo in the beginning of your commands. There are a lot of useful packages like docker, LAMP related packages from the mirrors of APK repositories. Feel free to explore them.

You can put all the books inside this machine and can treat this like a library. You can install apache2 package and set the configurations so that you can access those books (and can read from your web browser) very conveniently.

<ol style="background-color:#c9c9c9">
    <li>apk add apache2</li>
    <li>rc-service apache2 start</li>
</ol>


Note: In case if you end up messing something during the installation especially at the partition stage, reboot the machine and delete all partitions (If you don’t need any of the data in those partitions) and proceed try running the installation process again.

<b>Useful Links – </b>
* [Delete Partition Linux](https://phoenixnap.com/kb/delete-partition-linux)
* [SSH way to connecting to a machine](https://kb.iu.edu/d/aews#:~:text=SSH%20public%20key%20authentication%20relies,connect%20to%20the%20remote%20system.)
* [Alpine Post Installation](https://docs.alpinelinux.org/user-handbook/0.1a/Working/post-install.html)
* [Setup Alpine](https://docs.alpinelinux.org/user-handbook/0.1a/Installing/setup_alpine.html)
* [IPv4 Static Address Configuration](https://wiki.alpinelinux.org/wiki/Configure_Networking#IPv4_Static_Address_Configuration)
* [Directory Browsing](https://www.youtube.com/watch?v=NbWNPqPe6fU)