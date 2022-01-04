
# Building the Ultimate Ubuntu Desktop on a Pre-Made Gaming PC

#### Overview
Here we get you up and running with the latest desktop experience from Ubuntu, including step-by-step instructions for building the ultimate linux setup.

 - Choose an Ubuntu Desktop Version
 - Install Ubuntu Desktop from USB (without impacting Windows, if any)
 - Install PowerShell and VS Code using `apt-get`
 - Install and Configure `kvm` to run virtual machines
 - Configure daily backups to `LaCie` (or similar) USB drive
 - Install the Influxdata `TICK` stack for gathering and visualizing data

*TLDR: The objective of this write-up is to get you using linux for your day to day computing needs (and also learning to use powerful free tools for gathering and visualizing performance data).*

#### Fast Paths
For those wanting to build an entire Desktop system running Linux, you can follow every step. However, you can optionally skip to any task as desired, since the steps do not really depend on each other. For example, you do not need `powershell` or `vs code` for anything, but they are nice to have.  Also, I get you setup with `kvm` (to run virtual machines), but this is not needed for any future steps. A large focus will be on the "Performance" section.

Sections of this document:

- Hardware Setup
- OS Setup
- Ubuntu Desktop Basics
- Install PowerShell and Visual Studio Code
- General OS Section
- Virtualization Section
- Performance Metrics Collection and Data Visualization
- Appendix (Install InfluxDB 2.x instead of 1.x)

#### Hardware
Ideally, use a physical desktop for the most immersive experience.  This way you can easily add a dedicated SSD for the Ubuntu installation (though you could install onto existing disks that have Windows, etc.).

Here, we will assume a dedicated desktop exists, such as a mid-level gaming system (i.e. from BestBuy or similar). Then, basically we just install Ubuntu on it. I have done this build on`CyberPower` and `iBuyPower` Desktop PCs from BestBuy, though you could build your own or use anything really.

*Note: You can optionally run one or more of these steps on a virtual machine, but a desktop is recommended.*

#### Hard Drive Requirements
When you buy a pre-made system, also buy an extra hard drive so we can really dedicate our Linux install to it.

You need the following:

    1 or 2 TB SATA SSD (i.e. 2.5")
    SSD Mounting Kit (comes with tray, screws, etc.)
    Data cable for SSD to Motherboard connection
    Small Screw Driver

    *Note: You will also need to connect a power cable to the SSD, but that should be availble from your power supply.  Even the pre-mades will have these extra power cables by default, but they will never include the SSD data cable.*

#### Optional / Other Hardware

- Uninteruptable Power Supply such as `APC`
- Ethernet hub, if needed
- Network Cables, if needed
- Other nodes to monitor such as Raspberry Pi
- Other nodes to monitor such as `Akai MPC Live, X or One`

*Note: SSH access must be enabled on the `Akai MPC` so check out the excellent article at https://github.com/TheKikGen/MPC-LiveXplore/wiki/Enabling-SSH-on-the-MPC-Live-X-one-Force*



# Hardware Setup


*Note: This section requires a "helper" node (i.e. some PC or macOS) that can create a bootable USB thumb drive for BIOS updates (if desired)*

#### Build A Windows PC (and then install linux)

    #PREPARING THE SYSTEM
    - Use a pre-made gaming desktop or build your own
    - Connect an SSD to the system (will be used for Ubuntu install)
    - Purchase a LaCie USB Backup drive (we connect this later after the system is stable)
    - Purchase at least one USB thumb drive (ideally two)
    - Format a USB Thumb drive as fat32
    - Download BIOS updates for your motherboard and copy the image file to the top directory of the thumb drive
    - Place the USB thumb drive into the system
    - Power the system and enter the BIOS by pressing the approriate key (i.e. `DEL`, etc.)
    
*Tip: Optionally, reset the BIOS to optimized defaults and reboot before performing the BIOS update*

    - Then, navigate as needed to update the BIOS using the image on thumb drive
    - Finally, after the BIOS updates, allow the system to reboot
    
*Note: Optionally, enter BIOS again and ensure that CPU virtualization is active (or wait until the `kvm` section later in this write-up)* 


# OS Setup

*Note: This requires a "helper" node (i.e. some PC or macOS) that can create a bootable USB thumb drive for the Ubuntu install (using the `balena` application discussed below*

#### Download `Ubuntu Desktop`

    https://ubuntu.com/download/desktop

*Note: For developers you really should get version `20.04 LTS` which gives great stability and you can focus on coding.  We can optionally use something more recent such as the latest `21.10` which delivers more stable graphics, but impacts some apps such as `Unity3D` (if into that use `20.04 LTS` for sure).*

#### INSTALLING UBUNTU

    - Creating the USB requires a working system such as PC, Mac, etc.
    - If you have not already, download the desired Ubuntu release from https://ubuntu.com/download/desktop
    - Then, download and install the imager program Ubuntu recommends https://www.balena.io/etcher/
    - Choose a USB thumb drive you are okay with destroying (we will make it healthy again later)
    - Backup anything important on the thumb drive and then delete everything on it
    - Use the balena application to make the Thumb drive into a bootable USB by selecting the Ubuntu ISO when prompted)
    - Remove the USB thumb drive from your "helper" system
    - Place the USB thumb drive into your new system
    - Power on and follow the prompts to install Ubuntu 
    - When prompted for disk location, select the large SSD you installed
    - Enter a password to protect your disk with encryption when prompted for that option

*Note: The Ubuntu installer can only format and work with one disk at installation time.  This has the benefit of never wiping out your Windows installation (if any).*

#### OPTIONAL - REPAIR THE USB THUMB DRIVE

You may notice that your thumb drive that we turned into a bootable Ubuntu installer, may not appear as healthy on other systems.  This is a result of the balena program we used earlier. For smaller drives like 32GB usually they are not impacted, but if the drive is larger, it may appear and unformatted on your other systems. To resolve this, we can download a utility that will repair our USB thumb drive.

*Tip: Ignore the fact that this imager utility says "raspberry pi"; We use this just to format our USB thumb drive back to normal/healthy.*

Here is the utility:

 https://www.raspberrypi.com/software/


# Ubuntu Desktop Basics


#### The `Ubuntu` Boot Menu (`grub`)
Once `Ubuntu` has been installed, each time you power on, the Ubuntu `grub` boot menu will display for some seconds to allow us to arrow down to select Windows, or any other option that may be there.

#### Login to Ubuntu
Login with the user information you provided during setup.

#### Launch `Terminal`
Even though we have this beautiful Ubuntu desktop, most of our work early on will be from the `Terminal`.

#### Resizing Terminal
Press `CTRl` `SHIFT` `+` to increase the size
Press `CTRl` `-`         to decrease the size

#### Get Updates
You can follow the GUI prompts that will appear from time to time, or you can update from the command line using `apt-get`.

    #check for available updates
    sudo apt-get -y update

    #optional - show the updates
    sudo apt list --upgradable

    #install the updates
    sudo apt-get -y upgrade

*Note: Some commands we run in this setup guide will require `sudo` (elevated power).*

#### Show Video Card Model
We can see our video model using `lspci`.

    lspci | grep VGA

#### Example Output

    ike@ubuntu02:~$ lspci | grep VGA
    01:00.0 VGA compatible controller: NVIDIA Corporation TU116 [GeForce GTX 1660 Ti] (rev a1)

#### Optional - Use a "Proprietary" Graphics Driver
Assuming you have the right hardware, you can use the proprietary driver from Nvidia.  Ubuntu already knows the versions available from them and presents them as an easy to click radio button.

- Step 1 - Navigate to "Show Applications > SOftware & Updates > Additional Drivers"
- Step 2 - Observe that "Nouveaux" is currently checked
- Step 3 - Check the radio button at the top such as the `nvidia-driver-470 (proprietary, tested)`

#### Show Operating System Info

    cat /etc/os-release
    
#### Example Output
    
    mike@ubuntu02:~$ cat /etc/os-release 
    PRETTY_NAME="Ubuntu 21.10"
    NAME="Ubuntu"
    VERSION_ID="21.10"
    VERSION="21.10 (Impish Indri)"
    VERSION_CODENAME=impish
    ID=ubuntu
    ID_LIKE=debian
    HOME_URL="https://www.ubuntu.com/"
    SUPPORT_URL="https://help.ubuntu.com/"
    BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
    PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
    UBUNTU_CODENAME=impish
    mike@ubuntu02:~$ 

*Note: You can also use `uname -a` if you like that output.*

#### Quick `sync` with sudo
An old trick in linux is to `sync` before anything such as a reboot, etc.  The command flushes any transactions in flight to disk (and historically it is run twice).

While doing a `sync` is a nice thing, we can use it simply as something easy to type that will "freshen up" our `sudo` connection.

    sudo sync

*Note: After typing the above short command (or any sudo command), you will not be prompted for the sudo password again for some minutes.*

#### About `snaps` from `snapcraft.io`
Upon your first login to a fresh Ubuntu installation, you will be greeted with the "Ubuntu Software" application, which allows you to choose from many popular software packages to install as `snaps`.

These are pre-built apps that are ready to run, but might not be as powerful (or compatible) with your system as the official releases.

*Note: Sometimes `snaps` are made by community and sometimes by the official vendor so check the `Publisher` if interested to know.*

#### Install `gimp` as a `snap`
The application known as `gimp` is very popular photo editor for linux.  While it is not offically made by the author of the orginal, the `snap` is community supported and does the business.

    sudo snap install gimp

*Note: `snaps` can also be installed using the `Ubuntu Software` application.*

#### Create an Application Shortcut in Ubuntu
It is very useful to line up the taskbar with useful tools. You can right-click and create a shortcut by adding to favorites, which places the item on the taskbar for future use.

- Navigate to `Show Applications` at the bottom left of the Ubuntu desktop
- locate the icon for `Gnu Image Manipulation Program` (a.k.a. `gimp`)
- Right-click the icon and select `Add to Favorites`
- Press `ESC` on your keyboard to exit the `Show Applications` menu
- Now you can use the shortcut on the taskbar

#### Take a Screenshot with Ubuntu
Press `SHIFT` + `CTRL` + `PRINT SCREEN` on your keyboard and use the selection tool to choose the desired area.

#### Paste a Screenshot and export to `png`
- Open the `gimp` application
- Right-click the canvas and navigate to `Edit > Paste As > New Image` (or `Shift`+`Ctrl`+`v`)
- Optional - Edit the image as desired
- Press `CTRL`+`E` to export
- Name with `.png` extension or similar
- Press `CTRL`+ `q` to Exit gimp
- Optionally save or discard the `gimp` project

#### More about `snaps`
Working with `snaps` requires the `snapd` package, which will already be installed on Ubuntu by default. This is the backend for the `snap` command, which is how we installed `gimp` in a previous step.

You can see that `snapd` is installed with:

    sudo apt list snapd

#### List `snaps`
We can list a few snaps, showing only the main releases.

    sudo snap find --narrow

*Note: Releases with a checkmark indicate the release is from a verified snap publisher.*

#### Optional - Configure Backups to External USB Drive
Connect a `LaCie` (or similar) USB hard drive to your system.  Be sure to connect it directly to the PC and not to some extra device with USB ports.

Once connected you should be able to browse the device (no formatting needed). If this is a new drive, feel free to delete the default files they include (i.e. some windows and mac utils that we do not need)

Then simply lauch the native Ubuntu backup utilty `deja dup`, which is located in Applications.

*Note: On `Ubuntu 21.10` the LaCie drive quietly mounts without you having to see it on the desktop.*


# Install PowerShell and Visual Studio Code

*Note: You can install powershell and/or vs code as snaps, but using `apt-get` is more dependable currently as some snap versions (of powershell anyway) have issues with various Ubuntu releases (i.e. 21.04 hates powershell snaps).*

#### Optional - List your OS Info

    cat /etc/os-release

#### About PowerShell Installs - To `apt` or `deb`
You must have exactly version `18.04`, `20.04`, or `21.04` to get the bits from `apt-get`.

If you are running the latest `21.10` you cannot use `apt`, but instead should use `dpkg` as below.  Note that we use the old `20.04` version on Ubuntu `21.10`.

#### Install PowerShell for `Ubuntu 21.10`

    #get the bits
    wget -v https://github.com/PowerShell/PowerShell/releases/download/v7.1.5/powershell_7.1.5-1.ubuntu.20.04_amd64.deb
    
    #install the bits
    sudo dpkg -i powershell_7.1.5-1.ubuntu.20.04_amd64.deb
    
    #install dependencies
    sudo apt-get install -f

    #launch powershell
    pwsh


#### Legacy Install - PowerShell on `Ubuntu 21.04` or Older
The current install is the step we just performed above. However, if you are running `Ubuntu 21.04` or older, then you can simply use `apt-get` (instead of `dpkg`).

*Tip: Remember to `cat /etc/os-release` if you do not know your version number*

    #Optional - See the official Microsoft guide for `apt-get` at:
    https://docs.microsoft.com/en-us/powershell/scripting/install/install-ubuntu?view=powershell-7.1


    #Update the list of packages

        sudo apt-get update

    #Install pre-requisite packages

        sudo apt-get install -y wget apt-transport-https software-properties-common

    #pick only one of the following

        wget -q https://packages.microsoft.com/config/ubuntu/18.04/packages-microsoft-prod.deb
        wget -q https://packages.microsoft.com/config/ubuntu/20.04/packages-microsoft-prod.deb
        wget -q https://packages.microsoft.com/config/ubuntu/21.04/packages-microsoft-prod.deb

    # Register the Microsoft repository GPG keys
        
        sudo dpkg -i packages-microsoft-prod.deb

    # Update the list of packages after we added packages.microsoft.com

        sudo apt-get update

    # Install PowerShell

        sudo apt-get install -y powershell

    # Start PowerShell

        pwsh

#### Install Visual Studio Code using `apt-get`
    
    #From:
    https://code.visualstudio.com/docs/setup/linux

    #get the gpg key
    wget -qO- https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > packages.microsoft.gpg

    #install the gpg key
    sudo install -o root -g root -m 644 packages.microsoft.gpg /etc/apt/trusted.gpg.d/

    #add the repo
    sudo sh -c 'echo "deb [arch=amd64,arm64,armhf signed-by=/etc/apt/trusted.gpg.d/packages.microsoft.gpg] https://packages.microsoft.com/repos/code stable main" > /etc/apt/sources.list.d/vscode.list'

    #remove the gpg key (not needed since we added it already)
    rm -f packages.microsoft.gpg

    #install pre-requisites
    sudo apt install apt-transport-https

    #update package cache (now it will see vs code)
    sudo apt update

    #perform the install
    sudo apt install code

*Tip: Over time there will be updates available. You can ignore the prompts inside the app to download the update, and instead use `apt-get` as shown below*
 
    #Updating Visual Studio Code
    sudo apt-get -y update
    sudo apt-get -y upgrade

#### Add PowerShell Extension to VS Code
The `PowerShell` extension for VS Code adds automatic error checking to your powershell scripts (using `PSScriptAnalyzer`), among many other cool features.

    Step 1. From Visual Studio Code, acess the "Extensions" menu by pressing `CTRL`+`SHIFT`+`X`, or navigate to:

        Files > Preferences > Extensions

    Step 2.  Search for `powershell` and click install.
    
*Tip: Observe that you must click to "trust" the workspace or similar to run powershell. You'll get the hang of it.*



# General OS Section

#### Show Hidden Files in Ubuntu
Unrelated to any apps thus far, we now discuss some general os items.

Sometimes, documentation will something like we placed your stuff in `~/.config/` and you be like what?  Well, this cool feature will show you shose hidden directories.

    1. Locate the `Files` browser (an icon that looks like a folder on your Ubuntu taskbar)
    2. Press `CTRL+H` to show/hide any hidden files or folders.


#### Prevent OS Upgrade Prompts (for users of `Ubuntu LTS`)
This is valuable perhaps if you are running an `lts` release. By adding the following, we can prevent prompts to upgrade to anything that is not `lts`.

Simply edit the file below and change `Prompt=normal` to `Prompt=lts` near the bottom of the file.

    sudo nano /etc/update-manager/release-upgrades

*Note: If you are not running an `lts` release already, then the system assumes the setting to be `normal` even if you set it to `lts`.*

#### Show Hard Power Offs
Much like a check disk (or `chkdsk`) in Windows, Linux will run an `fchk` after a hard power off (i.e. loss of power, user held the power button down, etc.).  The following returns log entires related to `fchk` (both naturally occurring or forced `fchk` will show here).

    sudo journalctl -b --no-pager | grep systemd-fsck

#### Example output for the above

    mike@ubuntu02:~$ sudo journalctl -b --no-pager | grep systemd-fsck
    Nov 02 12:49:50 ubuntu02 systemd-fsck[1260]: fsck.fat 4.2 (2021-01-31)
    Nov 02 12:49:50 ubuntu02 systemd-fsck[1260]: There are differences between boot sector and its backup.
    Nov 02 12:49:50 ubuntu02 systemd-fsck[1260]: This is mostly harmless. Differences: (offset:original/backup)
    Nov 02 12:49:50 ubuntu02 systemd-fsck[1260]:   65:01/00
    Nov 02 12:49:50 ubuntu02 systemd-fsck[1260]:   Not automatically fixing this.
    Nov 02 12:49:50 ubuntu02 systemd-fsck[1260]: Dirty bit is set. Fs was not properly unmounted and some data may be corrupt.
    Nov 02 12:49:50 ubuntu02 systemd-fsck[1260]:  Automatically removing dirty bit.
    Nov 02 12:49:50 ubuntu02 systemd-fsck[1260]: *** Filesystem was changed ***
    Nov 02 12:49:50 ubuntu02 systemd-fsck[1260]: Writing changes.
    Nov 02 12:49:50 ubuntu02 systemd-fsck[1260]: /dev/sdb1: 12 files, 1337/130811 clusters
    Nov 02 12:49:50 ubuntu02 systemd-fsck[1261]: /dev/sdb2: recovering journal
    Nov 02 12:49:50 ubuntu02 systemd-fsck[1261]: /dev/sdb2: clean, 312/46848 files, 51367/187392 blocks
    Nov 02 12:50:24 ubuntu02 systemd[1]: systemd-fsckd.service: Deactivated successfully.
    mike@ubuntu02:~$ 

*Note: See the discussion at `https://askubuntu.com/questions/112907/where-are-fsck-results-logged-at-boot-time-after-forcefsck` to learn more.*

#### Optional - Add Networking Tools
This will allow us to use the `route` command to look at the routing table, etc.

    sudo apt install net-tools

#### Optional - Add Ethernet Networking
Connect an ethernet cable and configure an IP Address by clicking the icons at the top right of the desktop to get to networking.  I added an IPv4 address of `10.100.0.21` with a 24 bit mask of `255.255.255.0` and a gateway (which does not exist) of `10.100.0.1`.

However, I also have a wireless connection for internet, and I want that to take preference when looking things up. If we look at my routing table, we can see that the wireless connection has a lower priority than the new ethernet connection (the metric of the ethernet connection is lower so it has preference).

    #default - ethernet has metric of 100 and wireless has metric of 600
    mike@ubuntu02:~$ route
    Kernel IP routing table
    Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
    default         my.jetpack      0.0.0.0         UG    600    0        0 wlp4s0
    default         _gateway        0.0.0.0         UG    20100  0        0 enp3s0
    10.100.0.0      0.0.0.0         255.255.255.0   U     100    0        0 enp3s0
    link-local      0.0.0.0         255.255.0.0     U     1000   0        0 wlp4s0
    192.168.1.0     0.0.0.0         255.255.255.0   U     600    0        0 wlp4s0

    #add a route
    sudo route add -net 10.100.0.0 netmask 255.255.255.0 metric 720 dev enp3s0 
    
    #delete old route
    sudo route del -net 10.100.0.0 netmask 255.255.255.0  metric 100
    
    #show the updated routing table - now ethernet has a metric of 720
    mike@ubuntu02:~$ route
    Kernel IP routing table
    Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
    default         _gateway        0.0.0.0         UG    20100  0        0 enp3s0
    default         my.jetpack      0.0.0.0         UG    20600  0        0 wlp4s0
    10.100.0.0      0.0.0.0         255.255.255.0   U     720    0        0 enp3s0
    link-local      0.0.0.0         255.255.0.0     U     1000   0        0 wlp4s0
    192.168.1.0     0.0.0.0         255.255.255.0   U     600    0        0 wlp4s0



# Virtualization Section

#### Install KVM on Ubuntu 20.x
In the next few steps, we get configured to run virtual machines. This section is optional and we will not need this for future steps.

*Note: Below we follow steps the excellent guide about `kvm` at https://phoenixnap.com/kb/ubuntu-install-kvm`*

#### Confirm Hardware Supports Virtualization
BIOS support for virtualization started appearing around 2007 or so, though not all devices are guaranteed to have it.

    egrep -c '(vmx|svm)' /proc/cpuinfo

#### Install `cpu-checker`
In addition to supporting virtualization, the BIOS must also be configured for virtualzation.  Often this is already done for you, but we can check with the `kvm-ok` command, which is part of a package called `cpu-checker`.

    sudo apt install cpu-checker

*Note: After installing `cpu-checker` restart your terminal, and proceed with the next steps.*

#### Run the `kvm-ok` command
The `kvm-ok` command will tell you if your BIOS is setup peoperly.

    kvm-ok

#### example output from `kvm-ok`

    mike@ubuntu02:~$ sudo kvm-ok
    INFO: /dev/kvm exists
    KVM acceleration can be used

*Note: If you fail the `kvm-ok` test, shutdown and enter the BIOS to enable the required virtualization technology. After making changes in the BIOS, be sure to cold boot (full shutdown) for the changes to take affect. Then, try `kvm-ok` again as needed.*

#### Install KVM

    sudo apt install -y qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils

#### Optional - List Users
For some upcoming steps, we will need to choose one or more "users" on our system that will be allowed to run virtual machines. There are many commands for this such as `compgen -u` or `cat /etc/passwd`. Below is one example of listing users.

    #list users
    cut -d: -f1 /etc/passwd

#### Add Desired Users to `libvirt`
The `adduser` command below adds the desired user ("mike" in this case) to the `libvirt` group.  If you are already in the group, it will let you know.

    sudo adduser mike libvirt

*Note: If you performed the install as that user, you will already be in the `libvirt` group.*

#### Add Desired Users to `kvm`
Do not skip this step, since there will not be any users in this group yet.

The `adduser` command below adds the desired user ("mike" in this case) to the `kvm` group.

    sudo adduser mike kvm

#### Check Health of KVM

    virsh list --all

#### Check Service Status

    sudo systemctl status libvirtd

*Note: Press `q` to exit the service status view*

#### Optional - Start Service, If Needed
If the above showed that kvm was not running (unlikely), then you can start it with:

    #if not running
    sudo systemctl enable --now libvirtd

*Note: The above should not be needed in most cases since the installation takes care of this*

#### List Contents of `images` Folder
On a the new installation of `libvirt` we just performed, there will be no images here yet.

    sudo ls -lh /var/lib/libvirt/images/

#### Copy an ISO
Copy (`cp`) or move (`mv`) the desired ISO to the `images` directory.

    sudo mv /home/mike/Downloads/Windows.iso /var/lib/libvirt/images/Windows.iso

#### Install the `virt-manager` Application
Finally, we install the application that we can interact with to make and run virtual machines.

    sudo apt install virt-manager

#### Log Off
Save any work and log off / log on again. This will ensure that `virt-manager` runs smoothly without `sudo`.


#### Using the `vmm` Icon
From "Show Applications" at the bottom left of the Ubuntu desktop, locate the application called `vmm` (Virtual Machine Manager). Right-click and add to your favorites if desired.  Finally, click the icon.

*Note: If you get an error when launching the `vmm` application, be sure you have added your user to the appropriate groups we discussed (`libvirt` and `kvm`), and be sure you have logged off / logged on).*

#### Launching `virt-manager`
Optionally, we can run `virt-manager` as `sudo` (instead of using the `vmm` icon).  However, `sudo` should not be required.

    #if you must have sudo
    sudo virt-manager

#### Create a VM from GUI
Launch `virt-manager` as desired from terminal or application menu. Then, point to your ISO and create your VM.

*Note: *You can also create a VM from the command line, if desired.*


# Performance Metrics Collection and Data Visualization

*Note: This is the best part. If you get this up and running you can gather data points from anything, including system performance of windows, linux, macOS, Raspberry Pi, and even your drum machine (if your drum machine runs linux, like the Akai MPC Live, X or One)*


#### `InfluxDB` and the `TICK` Stack
Written in Google `go`, the products from the `influxdata` team are lightweight, powerful, and open source. Using their software we can collect and visualize performance data easily.

#### `InfluxDB` Versions
There are currently two version "trains" that are fully updated and maintained. This write-up focuses on the `1.x` train, but details of using `2.x` are listed in the Appendix at the end.

 Version 1.x - Available from `apt-get`, delivers `InfluxDB 1.8.x`, and associated `TICK` stack
 Version 2.x - Available via download of `dpkg`, delivers `InfluxDB 2.x` (but lots of changes too)

#### The `Tick` Stack
The `TICK` stack is `telegraf`, `influxdb`, `chronograf`, and `kapacitor`.

Basically, the database is `InfluxDB`, and `Telegraf` collects/sends the stats to `InfluxDB`.  Don't worry too much about the other two as they are tightly integrated and you will be guided when you need enter the user and password for initial setup, etc.

#### Install Order
You can install the entire `TICK` stack in one line (and in any order), but we will do them individually.

Typically, I start with `InfluxDB` and then move on from there.  First though, we need to add a `gpg` key for those that need it. We do that next.

#### Add the`gpg` key for `influxdb` (requird for `Ubuntu 20.04` or lower)
If running `Ubuntu 21.04` or later, you can skip this, but for `Ubuntu 20.04` we must add the influxdb `gpg` key in order to use `apt-get`.

Doing the following (for `Ubuntu 20.04` or older) enables us to download one or more `influxdata` products, since they all use the same `influxdb.key`.

    wget -qO- https://repos.influxdata.com/influxdb.key | sudo apt-key add -
    source /etc/lsb-release
    echo "deb https://repos.influxdata.com/${DISTRIB_ID,,} ${DISTRIB_CODENAME} stable" | sudo tee /etc/apt/sources.list.d/influxdb.list

*Tip: We can enter `cat /etc/os-release` from a terminal to see our version, if needed.*

#### Install `InfluxDB`
Now that we have the `gpg` key (if needed), we can refresh our available packages as shown below.  We can then install `influxdb` (or any other component of the `TICK` stack).

We will start with installing `InfluxDB`.

    #update the package list
    sudo apt-get update

    #install influxdb
    sudo apt-get install influxdb

    #use systemctl to `unmask` the service
    sudo systemctl unmask influxdb.service

    #use systemctl to `start` the service
    sudo systemctl start influxdb

*Note: The official InfluxDB installation instructions have us to `unmask` the service. This is likely to help issues experienced on older Ubuntu versions as documented in issue `https://github.com/influxdata/influxdb/issues/6253`.*  

#### Optional - Enable `ufw` firewall and allow port `8086`
By default, the Ubuntu Firewall (`ufw`) will be installed already.  However, it must be enabled and configured which we do below.  We also allow the default InfluxDB port of `8086` on the firewall.

    sudo apt-get install ufw

    sudo ufw enable

    sudo ufw allow 8086/tcp

*Note: The `ufw` firewall technique above is borrowed from `https://computingforgeeks.com/install-influxdb-on-ubuntu-and-debian/`*

#### Install `Telegraf`
Since we already have the gpg key (if needed), we can simply install `telegraf` using `apt-get`.

    sudo apt-get update
    
    sudo apt-get install telegraf

*Note: The official Telegraf instructions do not mention the need to `unmask` the service like we do for InfluxDB.*

#### Install `chronograf`
You can skip this if running `InfluxDB 2.x`, but for `1.x` we must install `cronograf` to manage `InfluxDB` users.

    sudo apt-get install chronograf

*Note: *You can access the `cronograf` web interface at `http://localhost:8888/`*

#### Install `kapacitor`

    sudo systemctl install kapacitor

#### Use `apt list` to show `TICK` stack
We can use `apt list` to show the components of the `TICK` stack, and determine if they are all installed.

    sudo apt list telegraf influxdb chronograf kapacitor | grep amd64

#### Example Output from above
    
    mike@ubuntu02:~$ sudo apt list telegraf influxdb chronograf kapacitor | grep amd64

    chronograf/unknown,now 1.9.1-1 amd64 [installed]
    influxdb/unknown,now 1.8.10-1 amd64 [installed]
    kapacitor/unknown,now 1.6.2-1 amd64 [installed]
    telegraf/unknown,now 1.20.3-1 amd64 [installed]
    mike@ubuntu02:~$ 

#### Backup the default Configurations
Before editing your default configuration files, you might want to back them up.

The following will make a copy of the config files into your `/home` directory.

    #backup the config for influxdb
    sudo cp /etc/influxdb/influxdb.conf ~/influxdb.conf.bak

    #backup the config for telegraf
    sudo cp /etc/telegraf/telegraf.conf ~/telegraf.conf.bak

    #backup the config for kapacitor
    sudo cp /etc/kapacitor/kapacitor.conf ~/kapacitor.conf.bak

*Note: Once you get configs you like then you can save your customized versions as backups somplace as well.*

#### Optional - Edit Configuration Files
Optionally, edit the configs as desired.

    #influxdb config
    sudo nano /etc/influxdb/influxdb.conf
    
    #telegraf config
    sudo nano /etc/telegraf/telegraf.conf

    #kapacitor config
    sudo nano /etc/kapacitor/kapacitor.conf

#### Tips for Using the `nano` Editor
To edit text files from terminal in Linux we can typically use `vi` or `nano`. Here we discuss `nano`.

    CTRL + W                       # where is something. Use this to search for text
    CTRL + O, Enter, CTRL + X      # save the file
    CTRL + X                       # exit without saving changes

#### Tips for Using the `vi` Editor
Some systems have both `nano` and `vi` and you can pick your favorite.  However, some only have `vi`.  In such cases it is good to be familiar with both. Here, we discuss `vi`.

    i      # press "i" to begin editing
    ESC    # press ESC to stop editing
    :wq!   # press ESC and then type :wq! (and hit enter) to write the file
    :q!    # exit without making changes

*Note: The `Akai MPC X, Live and One` only have the `vi` editor.*

#### About `InfluxDB` Configuration File
By default, `InfluxDB` reads in the config file and runs it using all defaults, which are listed but commented out.  Even though they are commented out, they are active and working configuration elements (behind the scenes). Also, you do not need to have all of the entries listed which are known defaults.  For example, you can make a config file with only the things you have tweaked, if desired.

The following is a snippet from the `influxdb` documentation:

> Settings that are commented out are set to the internal system defaults. Uncommented settings override the internal defaults. Note that the local configuration file does not need to include every configuration setting.

#### About `Telegraf` Configuration File
For the `Telegraf` configuration, you must "uncomment" anything you want to run.  By default, some things are already enabled for you (uncommented).*

#### Notable `InfluxDB` Config Setting `flux-enabled`
For more info see `https://docs.influxdata.com/influxdb/v1.8/flux/installation/#enable-flux`, but basically we can use `FluxQL` (an InfluxDB query language) if we set `flux-enabled` (in the http section of the co) to true. This is useful for building dashboards in web interface, etc.

    flux-enabled = true

#### Notable `telegraf` Config Settings
By default, a few things are already uncommented for us in the `telegraf` configuration (i.e. a couple of stats), but we need to point to InfluxDB, and also add any other metrics to track (i.e. by uncommenting anything interesting).

    [inputs.net]
    [inputs.influxdb]

#### Start the `influxdb` and `telegraf` Services
Reboot your system, or start the services as needed.

    #start influxdb
    sudo systemctl start influxdb

    #start telegraf
    sudo systemctl start telegraf

#### For InluxDB `1.8` create users manually using the `influx` CLI
Open a terminal and type `influx`

    influx
    
#### Example output from above
This example shows the interactive cli, known as `influx`.

    mike@ubuntu02:~$ 
    Connected to http://localhost:8086 version 1.8.10
    InfluxDB shell version: 1.8.10
    > 

*Tip: Enter lower case `l`. or upper case `L` to clear the screen, and type `quit` or `exit` to close the CLI and return to your shell*

#### Create `InfluxDB` users
Before we create a user for ourselves, we must first create the `admin` user and set its password. Here, we set the password to `mpc` (adjust as desired).

*Note: The password must be enclosed in single or double-quotes.*

    #create admin user, and set password to "mpc"
    CREATE USER admin WITH PASSWORD 'mpc' WITH ALL PRIVILEGES

Then we can create additional users; Here, we create a user called `mpc` and set the password to `mpc`.

    #create our main user, and set password to "mpc"
    CREATE USER mpc WITH PASSWORD 'mpc' WITH ALL PRIVILEGES

*Note: With the `influx` CLI, there will be no response when a command succeeds (only when it fails).*

#### Example output
Here I create the admin user and also another user.

    mike@ubuntu02:~$ influx
    Connected to http://localhost:8086 version 1.8.10
    InfluxDB shell version: 1.8.10
    > CREATE USER admin WITH PASSWORD 'InfluxSecret' WITH ALL PRIVILEGES
    > CREATE USER mpc WITH PASSWORD 'mpc' WITH ALL PRIVILEGES


#### Access the `Cronograf` Web Interface at port `8888`
We can use the `cronograf` web interface at port `8888` for creating and viewing dashboards, as well as the ability to manage users and databases for `InfluxDB`.  

    #for influxdb 1.x, we use cronograf at 8888 for most things
    http://localhost:8888

*Note: For `InfluxDB 2.x`, you can use the InfluxDB web interface at `8086`.*


#### Restart Services
Once you get some configs you like, such as monitoring your local system, etc. then be sure to restart the relevant service for your config file to take affect:

    #restart influxdb
    sudo systemctl restart influxdb

    #restart telegraf
    sudo systemctl restart telegraf

    #restart kapacitor
    sudo systemctl restart kapacitor

#### Stopping Services
Sometimes you may just want to `stop` a service instead of `restart`. For example, to run a one-liner that the InfluxDB web interface generates for you.

    #stop influxdb
    sudo systemctl stop influxdb

    #stop telegraf
    sudo systemctl stop telegraf

#### Start the Services

    #start influxdb
    sudo systemctl start influxdb

    #start telegraf
    sudo systemctl start telegraf

#### Service Status

    sudo systemctl status influxdb

    sudo systemctl status telegraf


#### Optional - Install the `influx` Extension for VS Code
Said to feel like javascript, the custom scripting language `influxQL` can be written using the `Cronograf` web interface, or you can add the Visual Studio Code extension called `influx`.

While the Influxdata products are written purely in google `golang` (a.k.a. `go`), this `InfluxQL` scripting language gives web developers (I am not one them) the ability to slice and dice data and visualize as desired using scripts.

    Step 1. From Visual Studio Code, acess the "Extensions" menu by pressing `CTRL`+`SHIFT`+`X`, or navigate to:

        Files > Preferences > Extensions

    Step 2.  Search for `influx` and click install.


#### Monitor other nodes with `telegraf agent`
Use the following instructions to monitor one or more additonal devices besides your desktop, such as a `Rapsberry Pi` or an `Akai MPC Live, X or One`.

Simply paste in the code below, or manually navigate to the influxdata site and select the desired version to download.

Here, we select `armv7` which works well for Rapspberry Pi and `Akai MPC Live, X or One`.

*Note: Many more architectures are supported, including the 64 bit `armv8`, but for `Akai MPC Live, X or One` be sure to use the `armv7` bits since they are 32 bit (like the `MPC`).*

For this we will install the `telegraf agent` on the remote node; Specifically this is a copy of folders not a real install. Specifically, we place the downloaded `telegraf` bits on the server or device to monitor and then `telegraf` uses its own folder structure (i.e. `/etc`, `/usr`, etc.).

As we know, `telegraf` can send stats locally or remotely to InfluxDB.  Since we do not want to run a bunch of software on every node we want to monitor, we can simply place this "agent" (`telegraf agent`) on the server or node to monitor and just run that one piece. Then, as the data collects (i.e. on our real `TICK` stack we deployed earlier) we can view the data in dashboards, etc.

- Option 1 - You can manually download `telegraf agent`, if desired.
     
     https://portal.influxdata.com/downloads/

- Option 2 - Use `wget`, which exists on most linux distributions (including the `Akai MPC Live, X and One`).

*Tip: Be sure to `cd` to the appropriate directory first as shown in the examples below*

    #linux example - place in your home directory
    cd ~
    wget https://dl.influxdata.com/telegraf/releases/telegraf-1.20.3_linux_armhf.tar.gz
    tar xf telegraf-1.20.3_linux_armhf.tar.gz

    #mpc example - place on USB or SD card, etc.
    cd /media/my-mpc-thumb-drive
    wget https://dl.influxdata.com/telegraf/releases/telegraf-1.20.3_linux_armhf.tar.gz
    tar xf telegraf-1.20.3_linux_armhf.tar.gz

- Option 3 - Place the bits on a USB thumb drive or SD card
    
    This allows you to download using your regular desktop and then insert the USB or SD into the device to monitor. You can run `telegraf` right from the USB.

#### About the `telegraf` Configuration File (`telegraf.conf`) 
After uncompressing the package with `tar` you can `cd` into the folder structure until you get to the `telegraf` folder.

    #change directory
    cd <path to telegraf folder>  # (i.e. cd /home/pi/telegraf-1.20.3/etc/telegraf)

    #edit config
    sudo nano telegraf.conf

#### What to Edit in `telegraf.conf`
With the configuration file open in `nano` (or similar), scroll down to your hostname.  Here you can optionally set a custom name for this host that will appear on your dashboards.  By default, it uses the system name if you do not specify anything here.

    #optional - configure hostname
    hostname = "pi"

Once done with the `hostname` settings (if any), scroll down to `OUTPUT PLUGINS`. Here, we configure `outputs.influxdb` so we can write to InfluxDB over the network.
    
    [[outputs.influxdb]]
    urls = ["http://10.100.0.21:8086"]
    database = "telegraf"
    timeout = "5s"
    username = "mpc"
    password = "mpc"
    user_agent = "telegraf"

Finally, search for the `INPUT PLUGINS` section. Here we can configure things to monitor by "uncommenting" (removing the pound sign / hash mark `#`) from the desired line.

*Note: By default, some things are already "uncommented" for us.*

    #already configured (no action required)
        inputs.cpu
        inputs.disk
        inputs.diskio
        inputs.kernel
        inputs.mem
        inputs.processes    #Returns “System - Total Processes”
        inputs.swap
        inputs.system

    #Configuration Required (i.e. uncomment these)

        inputs.kernel_vmstat
        inputs.net
        inputs.netstat       #Returns "System - Open Sockets” and “System - Sockets Created/Second”
        inputs.procstat      #Returns “Processes - Resident Memory (MB)” and “Processes – CPU Usage %” (Requires adding name of thing to track)
        inputs.wireless      #If your device has wireless

#### `inputs.influxdb` vs `outputs.influxdb`
When just running the `telegraf agent` on a node, we can ignore `inputs.influxdb` if you see that; This is because we're only interested in the `outputs.influxdb` (which we configured earlier).

#### Some other Configuration Settings
These look fun to test. Add your own findings as you rummage through the config file.

    #TODO / Research / OTHER

        inputs.github
        inputs.internet_speed
        inputs.linux_sysctl_fs
        inputs.openweathermap
        inputs.smart
        inputs.socket_listener
        inputs.sysstat
        inputs.systemd_units
        inputs.temp


#### Run `telegraf` binary and get `--help`
To actually run `telegraf` agent on some node, we need the full path to the binary, or copy it someplace you like.

    /home/pi/telegraf-1.20.3/usr/bin/telegraf --help

#### Run `telegraf` binary with `telegraf.conf`
Adjust for your paths as needed and simply point `telegraf` to the `--config` file location.

    /home/pi/telegraf-1.20.3/usr/bin/telegraf --config /home/pi/telegraf-1.20.3/etc/telegraf/telegraf.conf

#### Example Output

    pi@stats001:~/telegraf-1.20.3 $ /home/pi/telegraf-1.20.3/usr/bin/telegraf --config /home/pi/telegraf-1.20.3/etc/telegraf/telegraf.conf
    2021-10-29T18:20:54Z I! Starting Telegraf 1.20.3
    2021-10-29T18:20:54Z I! Loaded inputs: cpu disk diskio kernel mem net processes swap system wireless
    2021-10-29T18:20:54Z I! Loaded aggregators: 
    2021-10-29T18:20:54Z I! Loaded processors: 
    2021-10-29T18:20:54Z I! Loaded outputs: influxdb
    2021-10-29T18:20:54Z I! Tags enabled: host=pi
    2021-10-29T18:20:54Z I! [agent] Config: Interval:10s, Quiet:false, Hostname:"pi", Flush Interval:10s

*Note: Press `CTRL+C` to exit to the running process and stop the stats collection.*

#### Look at Dashboards
From the machine that is running your `TICK` stack, navigate to `http://localhost:8888` (or similar) to reach the `Cronograf` web interface. From there, click on the `hosts` list (looks like an "eye") in the left pane.  In the right pane, observe that the system we just set up shows there.  Also, a dashboard was created for you automatically using some of the collected stats.

#### Next Steps, Other Stats
Spice up your dashboards with some other fun stats.

    #get stats related to space (i.e. # of people in space, etc.)
    http://open-notify.org/

    #get stats about weather
    https://github.com/influxdata/telegraf/tree/master/plugins/inputs/openweathermap

*Note: To try out the weather stats, you need to set up a free API account at the openweathermap site. Also be sure to set your charts to last 6 hours when troubleshooting since their API does not report as often as it should (also you may have to restart telegraf when the weather api times out. Not perfect, but does work for occasional free weather).*

#### Use `InfluxQL` to grab weather reports
Here is an example of some `InfluxQL` that you can write yourself or have auto-generated for you.  This was done at the `8888` port using the web interface (the normal place you are working on things with `TICK` stack 1.x).

Along with the proper telegraf configuration and signing up for a free api account at `https://openweathermap.org/`, I can track weather stats from my Raspberry Pi (or any node with internet access).

```
  from(bucket: "telegraf/autogen")
  |> range(start: v.timeRangeStart)
  |> filter(fn: (r) => r._measurement == "weather")
  |> last()
```

*Note: The above assumes you have already edited your `telegraf.conf` to customize the `openweathermap` section.*


#### Optional - Monitoring a UPS (i.e. `APC`)
You can get an "Uninteruptable Power Supply" (such as `APC`) from BestBuy or similar.  They are really great to have. These can deliver stable power to your devices even when power goes down (at which point it switches to battery power for some time).

The very popular model known as `APC` can be monitored from `telegraf` using a pre-made plugin `app` available in the `telegraf.conf`.

Once up and running, `telegraf` can then gather metrics from your `APC`.

*Note: This requires ethernet connectivity to the `APC` (i.e. you cannot use USB for this).*

Step 1. - First, you need to edit your `telegraf` configuration to uncomment `[[inputs.apcupsd]]`.

Step 2. - Then adjust the default address, if needed.

    tcp://127.0.0.1:3551



# Appendix - Most Should Skip This Unless Wanting InfluxDB 2.x


#### Learn More About InfluxDB
As we know, there are two code trains (`1.x` and `2.x`), which are both maintained and considered "latest". The two versions can run side by side, but for simplicity the recommendation is to run `2.x` on another machine.

*Tip: When your system runs InfluxDB `1.x`, be sure to ignore the prompts to look at `2.x` documentation.*

    #influxDB 1.8
    https://docs.influxdata.com/influxdb/v1.8/administration/authentication_and_authorization/#authorization

    #influxdb 2.x
    https://docs.influxdata.com/influxdb/v2.0/get-started/
    
#### Optional - Installing `InfluxDB 2.x`
Not needed if you have the `TICK` stack up and running with `InfluxDB 1.x` already.

However, for advanced users, you can download the bits for 2.x, using the "download button" at the following site.

    #Download influxdb 2.x for (choose the `amd64` link)
    https://docs.influxdata.com/influxdb/v2.0/install/?t=Linux

*Note: We select `amd` even when running on an Intel systems.  This is just how the naming works.*

    #Unpackage contents to the current working directory
    tar xvzf path/to/influxdb2-2.0.9-linux-amd64.tar.gz

    #Copy the influx and influxd binary to your $PATH
    sudo cp influxdb2-2.0.9-linux-amd64/{influx,influxd} /usr/local/bin/

*Note: If running side-by-side with version `1.x` then rename the v2 binaries, or run them from another directory.*

#### The InfluxDB `2.x` Web Interface
For `InfluxDB 2.x`, we can use the web interface at port `8086`. This interface is re-born again for `InfluxDB 2.x`.

*Note: The `InfluxDB` web interface was previously deprecated in `InfluxDB 1.3`, even though it was amazing.*

Upon your first login to `InfluxDB 2.x` you will be greeted with some "on-boarding" which will help you to create a user, Organization, Bucket, etc. You can simply make some names up since you can always add or change things later.

    #for influxdb 2.x, use the InfluxDB web interface
    http://localhost:8086


#### API Key
For `InfluxDB 2.x`, while working on the web interface, you may be given an API key that you can copy to your clipboard.

You can later paste that API key into your `telegraf` configuration file to allow easy access to `InfluxDB`.

*Note: In `InfluxDB 1.x` you can also enable `v2 auth`, which allows use of API keys.  However, user and password is the nice and easy way for lab on version `1.x`*

