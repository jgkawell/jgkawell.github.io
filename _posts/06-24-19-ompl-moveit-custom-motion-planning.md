---
title: Installing OMPL/MoveIt! for custom motion planning
date: 2019-06-24 12:00:00 -500
categories: [development, tutorial]
tags: [ros,moveit,motion planning,robotics]
---

## Introduction

I am in the process of creating a customized version of an OMPL planner that uses a personalized costmap to help robots learn to move around people in a more comfortable way. In order to do this, I needed a development environment that included ROS, OMPL, and MoveIt!. Specifically, I needed both OMPL and MoveIt! to be installed from source in order to augment their code so that I could add my customizations to OMPL and have MoveIt! use my custom planner within OMPL.

I naively thought at first that setting up this development environment would be easy. I found this handy tutorial on web.archive.org (the fact that it's archived should have been a red flag to me) and attempted to follow its super basic instructions. Turns out that OMPL and MoveIt! have some dependencies that must be installed in a particular way in order for everything to install properly. Through a lot of hacking, installing, reinstalling, and much head-banging, I finally figured out an installation process that leads to (what seems to be) a fully functional development environment for OMPL and MoveIt!.

I have worked out a couple methods to getting this working:

- This is a simple shell script I wrote that installs everything you need on a clean Ubuntu system.
- **(preferred)** This method builds a container in Docker.


NOTE #1: I am not an experienced C++ developer and so there may be inaccuracies in my explanation below and my methodology may be hacky/downright wrong. I welcome any and all comments to improve this system.

NOTE #2: If you want to develop in ROS but don't necessarily need custom installs of OMPL or MoveIt! and also happen to use Windows, I've written up a couple tutorials showing how to get ROS working on Windows using either WSL [here](https://jack-kawell.com/2019/06/24/setting-up-a-ros-development-environment-in-windows/) or Docker [here](https://jack-kawell.com/2019/09/11/setting-up-ros-in-windows-through-docker/).

## Installation (Ubuntu Desktop)

Before we begin, a couple things to note. First, this process should work well on a relatively clean version of Ubuntu 16.04 desktop. I would suggest using a VM (or [WSL](https://docs.microsoft.com/en-us/windows/wsl/about) first as it is guaranteed to be completely clean and so without any conflicting dependencies (the bane of my existence while figuring all this out).

Now, open a terminal window and download the install script from my GitHub: 

```bash
wget https://raw.githubusercontent.com/jgkawell/docker-scripts/master/Linux/ompl-install.sh
```

NOTE: The contents of this file are explained in full below if you want to know what exactly it does.

You may need to make the file executable:

```bash
chmod +x ompl-dev-install.sh
```

And finally, simply run the script and wait a while (the length will depend on the speed of your machine and internet connection): 

```bash
./ompl-dev-install.sh
```

And that should be it! Keep reading below if you'd like a full explanation of the installation script though

## Installation (Docker)

This method will set up everything you need for development within a Docker container. This is my preferred method as it is super repeatable and can easily be nuked and rebuilt without any hassle. It does require a tad bit more initial setup though.

### X server (GUI forwarding)

Now before we get started, we need to first set up the X-server on the host in order to pipe through GUI applications (think `rqt_console`, `rviz`, etc.). This is done differently if you're on a Windows or Linux host.

#### Windows Host

If you're running Windows, I'd suggest installing[VcXsrv](https://sourceforge.net/projects/vcxsrv/) and then I have a pre-built configuration file that you can simply run to have the X server ready to go (it's under `docker-scripts/Windows` in [this git repository](https://github.com/jgkawell/docker-scripts).

In addition to installing and setting up your X-server application, you'll also need to set the `DISPLAY` environment variable so that the Docker container knows where to push GUI applications. For that, if you're on Windows, run the below command in Powershell and remember to fill in your correct IP address (IP can be found in Settings -> Network & Internet -> Wi-Fi or Ethernet -> Click on the connection -> IPv4 address):

```powershell
[Environment]::SetEnvironmentVariable("DISPLAY", "{MY_IP_ADDRESS}:0.0")
```

NOTE: This variable "unsets" itself for some reason if you change consoles or restart. You can check to see if this variable is still set by using the command `Get-ChildItem Env:`.

#### Linux Host

If you're running  a Linux host... *{coming soon}*

### Startup

Now that our X server is setup, [install Docker](https://docs.docker.com/v17.12/install/) and then open a terminal/Powershell window and clone my git repository to get the necessary Docker files:

```bash
git clone https://github.com/jgkawell/docker-scripts.git
```

Now move into the `ompl` directory within the repository:

```bash
cd docker-scripts/ompl
```

And now we can run our favorite command to launch the system:

```bash
docker-compose up
```

This will take a while to run, but at the end it will have built an image called `jgkawell/ompl-dev:basic` and brought up a container called `ompl-dev-basic`. You can then ssh into the container with the typical command:

```bash
docker exec -it ompl-dev-basic bash
```

And from there you should have everything you need good to go. The container has tmux built in for multi-pane support (you'll notice there's a tmux config file that is copied into the container and sets up some of the most commonly used features like mouse support) as well as GUI forwarding. Go ahead and run through some [ROS](http://wiki.ros.org/ROS/Tutorials) and [MoveIt!](http://docs.ros.org/kinetic/api/moveit_tutorials/html/index.html) tutorials to check things out and happy hacking!

## Installation (shell script overview)

Here is the installation script in full. The `Dockerfile` and `docker-compose.yaml` are very similar to the below code so if you're wondering how those work you can look here for answers. Note that the sequence of installation is very important as each install requires one or more of the previous libraries to either install or function properly.

#### Install misc needed packages

```bash
# Install misc needed packages
sudo apt -y update
sudo apt -y upgrade
sudo apt -y install git build-essential cmake wget
```

This section just installs some basic packages we'll need for the installation of everything below.

NOTE: In a previous version of this post I had custom installations of `libccd`, `fcl`, and `octomap`. Turns out these are unnecessary as MoveIt! will install the needed versions for our source OMPL install.

#### Install ROS


```bash
# Install ROS (https://ros.org/install):
sudo sh -c 'echo "deb http://packages.ros.org/ros/ubuntu xenial main" > /etc/apt/sources.list.d/ros-latest.list'
sudo apt-key adv --keyserver 'hkp://keyserver.ubuntu.com:80' --recv-key C1CF6E31E6BADE8868B172B4F42ED6FBAB17C654
sudo apt -y update
sudo apt -y install ros-kinetic-desktop
sudo rosdep init && rosdep update
echo "source /opt/ros/kinetic/setup.bash" >> ~/.bashrc
sudo apt -y install python-rosinstall python-rosinstall-generator python-wstool
```


Anyone familiar with ROS will recognize these steps. One little change I took was to replace the `lsb_release` call with simply defining our version as `xenial`. The reason being that I didn't want to install the `lsb-release` package and this script is only designed to work with 16.04/Kinetic anyway. Also you'll notice that I install the `desktop` version of ROS and **not** `desktop-full`. This means that MoveIt! won't be installed as a package and we can install it hassle free from source below. Also, MoveIt! will install all the needed GUI tools for robot simulation so installing them with `desktop-full` is unnecessary. 


#### Install MoveIt!


```bash
# Install MoveIt! (https://moveit.ros.org/install/source/):
rosdep update
sudo apt -y install python-catkin-tools clang-format-3.9
cd ~/
mkdir ws_moveit
cd ws_moveit
wstool init src
wstool merge -t src https://raw.githubusercontent.com/ros-planning/moveit/master/moveit.rosinstall
wstool update -t src
rosdep -y install --from-paths src --ignore-src --rosdistro kinetic
catkin config --extend /opt/ros/kinetic --cmake-args -DCMAKE_BUILD_TYPE=Release
config --blacklist \
    moveit_chomp_optimizer_adapter \
    moveit_planners_chomp \
    chomp_motion_planner
catkin build
echo "source ~/ws_moveit/devel/setup.bash" >> ~/.bashrc
```


This installation is taken directly from the link in the code. You'll notice that I don't setup `ccache` and I add `chomp` to the blacklist but other than that this is pretty much just as it is in the MoveIt! tutorial.


#### Setup OMPL


```bash
# Install OMPL (http://ompl.kavrakilab.org/installation.html):
sudo apt -y install pkg-config libboost-serialization-dev libboost-filesystem-dev libboost-system-dev libboost-program-options-dev libboost-test-dev libode-dev
cd ~/ws_moveit/src
git clone https://github.com/cairo-robotics/ompl.git
cd ompl
git checkout tags/1.2.3
```


This section simply installs a bunch of needed packages for OMPL and clones the source code into our MoveIt! workspace. Note that this clones the CAIRO fork of OMPL. This isn't necessary unless you're going using my technique for creating custom cost functions in OMPL spelled out in more detail here. It also makes sure to checkout tag `1.2.3` as this is the OMPL version for ROS Kinetic.

```bash
# Make some changes to build files so OMPL will be found by MoveIt!
cd ~/ws_moveit
sed -i "s|DESTINATION \${CMAKE_INSTALL_LIBDIR}|DESTINATION /opt/ros/kinetic/lib/x86_64-linux-gnu |g" ./src/ompl/src/ompl/CMakeLists.txt
sed -i "s|\${catkin_INCLUDE_DIRS}|\${OMPL_INCLUDE_DIRS} |g" ./src/moveit/moveit_planners/ompl/CMakeLists.txt
sed -i "s|\${OMPL_INCLUDE_DIRS})|\${catkin_INCLUDE_DIRS}) |g" ./src/moveit/moveit_planners/ompl/CMakeLists.txt
# MoveIt! needs the package.xml file in order to locate OMPL for building
cd ~/ws_moveit/src/ompl
wget https://raw.githubusercontent.com/ros-gbp/ompl-release/debian/kinetic/xenial/ompl/package.xml
```


In order for everything to build properly, we need to edit a couple files in the workspace. The first one tells OMPL to install itself over the default OMPL that is installed with MoveIt! ([source](https://answers.ros.org/question/296238/custom-state-sampler-in-moveit/)). The second one tells MoveIt! to look for OMPL before catkin directories ([source](https://moveit.ros.org/install/source/dependencies/)). And finally we have to download the `package.xml` into the OMPL directory so that MoveIt! will include it in the build process (<a rel="noreferrer noopener" aria-label=" (opens in a new tab)" href="https://moveit.ros.org/install/source/dependencies/" target="_blank">source</a>).

#### Install OMPL

```bash
# Finally install MoveIt! with our source OMPL
cd ~/ws_moveit
sudo catkin clean -y
sudo catkin build # sudo is needed since we install OMPL in /opt
source ~/.bashrc
```

Finally the real reason that we're here! At first I thought you had to install OMPL according to the instruction on [their site](https://ompl.kavrakilab.org/). But it turns out that you can install it alongside MoveIt! and it "just works". First we clean the workspace and then we build it. I added in a re-sourcing of the `.bashrc` file just for good measure.

## Conclusion

So there it is. It took me weeks to finally figure out this particular solution and I'm posting it here to hopefully save some others the time and hassle I went through. Again I want to reiterate that this is simply a functional solution and definitely not optimal. If anyone reads this and has suggestions on how to slim this down or decrease compile time (I'm considering packaging this up as a Docker image on Docker Hub so that install faster) I'd welcome any and all feedback. Please leave them in the comments below.
