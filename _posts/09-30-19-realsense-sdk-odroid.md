---
title: Installing RealSense SDK on an ODROID-XU4
date: 2019-09-30 12:00:00 -500
categories: [development, tutorial]
tags: [ros, robotics]
---

In this post I'll walk through how to get the RealSense SDK up and running on the ODROID-XU4. This might seem trivial at first, but the steps can be a bit confusing.

For this tutorial I'll assume you're running Ubuntu 16.04 MATE with the 4.14 kernel on your ODROID. You can get this image from their website <a rel="noreferrer noopener" aria-label="here (opens in a new tab)" href="https://odroid.in/ubuntu_16.04lts/" target="_blank">here</a> and flash it to your ODROID's storage using Baleena Etcher <a href="https://www.balena.io/etcher/">here</a>.

As long as you've got that working, let's proceed. I'm basically going to run through the instructions that Intel provides on their GitHub <a href="https://github.com/IntelRealSense/librealsense/blob/master/doc/installation.md">here</a>, but with a slight modification for the d435i. All that said, the setup is pretty straight forward though the actual install itself may take a while since the ODROID isn't the fastest thing.

## Getting started

First, we need to make sure everything on our system is up to date. To do that run:

```bash
sudo apt-get update && sudo apt-get upgrade && sudo apt-get dist-upgrade
```

For the following steps you'll need `git` and `gedit` so let's install those now since they don't come preloaded on the ODROID's OS:

```bash
sudo apt install git gedit
```

Now go to a directory where you want to clone your repositories and then pull down their Git repo:

```bash
git clone https://github.com/IntelRealSense/librealsense.git
```

Navigate to the `librealsense` root directory before running the following commands:

```bash
cd librealsense
```

And now we'll install all the development tools needed to build the package:

```bash
sudo apt-get install git libssl-dev libusb-1.0-0-dev pkg-config libgtk-3-dev libglfw3-dev g++ build-essential cmake
```

We also need to run a permissions script which will setup the proper `udev` rules. Run this below:

```bash
./scripts/setup_udev_rules.sh
```

## Patching for ODROID & d435i

Now an important step for the ODROID with the 4.14 kernel is to run a patch provided by Intel. This is included in the repo you cloned before so simply run this command:

```bash
./scripts/patch-realsense-ubuntu-odroid.sh
```

Now before we build the package, we must first make a slight change to the source code to make it compatible with the d435i. This is due to the fact that the d435i has an integrated IMU which doesn't work with the ODROID and causes the device to fail on connection. To make this change we'll run the following command to edit the problem file:

```bash
gedit src/ds5/ds5-factory.cpp
```

Now in the gedit window scroll way down to the line that says `all_sensors_present &= (hids.capacity() &gt;= 2);` and change the `2` to `1`. And that's it! Save the file and close the gedit window to return back to your terminal window.

## Building and Installing

Now we're ready to finally build and install the package. From the root of the `librealsense` repository, run the command below to make a `build` directory and navigate to it:

```bash
mkdir build && cd build
```

Now we'll run our `cmake` command to set everything up with a couple flags:

```bash
cmake ../ -DBUILD_EXAMPLES=true -DBUILD_GRAPHICAL_EXAMPLES=false
```

Now is the big moment! We're finally going to build and install the `librealsense` package. Below I have the `make` command running in a single thread which runs <strong>very</strong> slowly, but I had issues with the ODROID locking up if I tried to multithread the build (I'm guessing this is due to RAM limitations on the board). If you want to, go ahead and try multithreading it by adding the flag`-j#` (where you can replace the `#` to any number of threads you want) to the solo `make` command.

```bash
sudo make uninstall
make clean
make
sudo make install
```

## Conclusion

If everything went well you should now be able to reboot your machine (`sudo reboot now`) and then plug in your d435i and it should all work. Try running the command `rs-color` and you should see a bunch of output demonstrating that the ODROID is now reading input from the Realsense.

If you have any issues with the above steps double check what I'm doing with the commands from Intel <a href="https://github.com/IntelRealSense/librealsense/blob/master/doc/installation.md">here</a> and <a href="https://github.com/IntelRealSense/librealsense/blob/master/doc/installation_odroid.md">here</a> and then let me know how I can update this to make it better. Hope this helps!
