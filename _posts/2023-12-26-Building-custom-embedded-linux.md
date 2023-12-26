---
layout: post
author: jack
category: [linux,hardware]
---

I recently started understanding the software part of embedded systems. And this is the first attempt to gets hands-on building custom linux-based images. 

### Yocto Project

#### Instructions to getting-started on building custom embedded linux -

1. Clone the Poky Github repo from Yocto with branch selected as the version of interest.
```
git clone -b kirkstone git://git.yoctoproject.org/poky
cd poky
```

2. Source the build environment script to create config template files and the build directory - 
`source oe-init-build-env`

3. Edit the `conf/local.conf` file if wanted to change the targeted machine name. By default, `qemux86-64` is selected.

4. Now run `bitbake core-image-sato`. This is going to take a lot of time and if everything went well, your `build/` directory will be populated with a variety of folders and the images are present inside the `/tmp/deploy/images/<MACHINE>`

5. To run the image that you just built, run `runqemu qemux86-64` and a new xterm window will open with the Yocto custom OS loaded. (note: mine was buggy for some reason. I wouldn't navigate after some time and all the icons were weird but I proceeded further anyways following the same procedure to create a custom raspberrypi image).



#### Instructions to create a quick base raspberrypi image for a targetted raspberrypy board - 

1. Follow the same first step above and clone the poky repo if not present, else skip it.

2. Now download the repo containing the meta data(image) of the raspberrypi for the same poky version (mine was "kirkstone") - `git clone https://git.yoctoproject.org/meta-raspberrypi/ -b kirkstone`

3. Now source the build environment script to create config template files just like previous case - `source oe-init-build-env`

4. Edit the `conf/local.conf` to add the machine name `MACHINE  ??= raspberrypi4-64` and `IMAGE_FSTYPES = "tar.xz ext3 rpi-sdimg"`. 

5. Also edit the `conf/bblayers.conf` to add the full path of the above cloned `meta-raspberrypi` github repo to the `BBLAYERS` variable in the end.  

6. Once done with the edits, go ahead and build the image using the command `bitbake rpi-test-image`. Again, this will take good amount of time to build the `.rpi-sdimg`. 

7. If the above step completed without any errors, now you can flash this image into an SD card and insert in the raspberrypi slot. 

8. To flash the image from the terminal, you can use `dd`. Below the full command
`sudo dd if=<path to the .rpi-sdimg file> of=/dev/sdaX` where X stands for the number designated for the (sd card) device on your machine.

9. Once done, connect your raspberrypi to a monitor, a keyboard and insert this sd card for the raspberrypi to boot from. 
