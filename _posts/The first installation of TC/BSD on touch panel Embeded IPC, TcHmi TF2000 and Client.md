---
title: 'The first installation of TC/BSD on touch panel Embeded IPC, TcHmi TF2000 and Client'
date: 2024-04-20
permalink: /posts/2024/04/20/tc-bsd-on-touch-panel-embeded-ipc-and-tchmi-tf2000/
tags:
  - freebsd
  - TcBSD
  - TcHMI
  - enigneering
  - TF2000
  - TwinCAT/BSD
---

````markdown
---
layout: post
title: "Setting up TwinCAT/BSD on CP6706 with GUI and TF2000"
tags: [TwinCAT, FreeBSD, BSD, CP6706, HMI]
---

## Hardware Info

| Specification | Details |
| :--- | :--- |
| **Model** | CP6706-0001-0050 |
| **Display** | 7" 800 x 480 |
| **Touch screen** | Single-finger touch screen |
| **Processor** | Intel Atom® E3827, 1.75 GHz, 2 core |
| **Memory** | 8GB DDR3L RAM |
| **Storage** | 40 GB CFast |
| **Interfaces** | 4 x USB 2.0, 1 x DVI |
| **Power supply** | 24 V DC |

## Install X Environments

First, we need to install the Xorg window system and configure the drivers.

```shell
# Install Xorg
$ pkg install xorg

# Add a user into video user group
$ doas pw groupmod video -m Administrator

# Get VGA info
$ pciconf -lv|grep -B4 VGA
# returns: "Intel"...

# Get input devices info (In X terminal)
$ doas libinput list-devices 
# returns: ELO Touch input device "dev/input/event3"
````

> **Note:** Pass the generated hardware info (VGA and Input devices) into the config files in the later steps.

## Install GUI (Windows Manager)

We will use **Openbox** as the window manager and **LightDM** as the display manager.

```shell
# Unblock FreeBSD repository
$ doas ee /usr/local/etc/pkg/repos/FreeBSD
# >> Change to: FreeBSD: { enabled: yes }

# Install openbox
$ doas pkg install openbox

# Install lightdm and greeter
$ doas pkg install lightdm lightdm-gtk-greeter

# Install on-screen keyboard to input password at hello screen
$ doas pkg install florence
```

### Configure LightDM

Edit the `lightdm.conf` to disable the cursor (optional for touch) and configure the greeter.

```shell
$ doas ee /usr/local/etc/lightdm/lightdm.conf
# >> xserver-command=X -nocursor

$ doas ee /usr/local/etc/lightdm/lightdm-gtk-greeter.conf
# >> keyboard= florence --focus
# >> keyboard-position= "75%,center -0;50%, 25%"
```

## Config Files

Below are the specific configurations required for the CP6706 hardware.

### Directory: `/usr/local/etc/X11/xorg.conf.d/`

**10-intel.conf.txt**

```text
Section "Device"
        Identifier      "Card0"
        Driver          "modesetting"
EndSection
```

**[10-monitor.conf.txt](https://github.com/KowalskiPi/TcBSD-CP6706-config/blob/main/_usr_local_etc_X11_xorg.conf.d/10-monitor.conf.txt)**

```text
Section "Monitor"
        Identifier      "HDMI-2"
        VendorName      "ELOTouch"
        ModelName       "Monitor0Model"
        Option          "DPMS" "false"
EndSection

Section "Screen"
        Identifier      "Screen0"
        Device          "Card0"
        Monitor         "HDMI-2"
        SubSection      "Display"
        Modes           "800x480"
        EndSubSection
EndSection
```

**[30-touchpad.conf.txt](https://github.com/KowalskiPi/TcBSD-CP6706-config/blob/main/_usr_local_etc_X11_xorg.conf.d/30-touchpad.conf.txt)**

```text
Section "InputClass"
        Identifier "touchpad"
        MatchIsTouchPad "on"
        Driver  "libinput"
        Option  "Tapping" "on"
        Option  "NaturalScrolling"      "on"
EndSection
```

**[40-touchscreen.conf.txt](https://github.com/KowalskiPi/TcBSD-CP6706-config/blob/main/_usr_local_etc_X11_xorg.conf.d/40-touchscreen.conf.txt)**

```text
Section "InputClass"
        Identifier      "touchScreen"
        MatchIsTouchscreen      "on"
        Driver  "libinput"
        Option  "Device" "/dev/input/event3"
EndSection
```

**[90-fonts.conf.txt](https://github.com/KowalskiPi/TcBSD-CP6706-config/blob/main/_usr_local_etc_X11_xorg.conf.d/90-fonts.conf.txt)**

```text
Section "Files"
        FontPath "/usr/local/share/fonts/urwfonts/"
        FontPath "/usr/local/share/fonts/TrueType/"
EndSection
```

### System Configuration

**[/boot/loader.conf](https://github.com/KowalskiPi/TcBSD-CP6706-config/tree/main/boot)**

```properties
kern.geom.label.disk_ident.enable="0"
kern.geom.label.gptid.enable="0"
cryptodev_load="YES"
zfs_load="YES"
hint.attimer.0.clock="0"
cuse_load="YES"
hw.psm.synaptics_support=1
usbhid_load="yes"
hw.usb.usbhid.enable=1
kldload i915kms
hw.vga.textmode=1
fdescfs_load="yes"
```

**[/etc/fstab](https://github.com/KowalskiPi/TcBSD-CP6706-config/tree/main/etc)**

```text
# Device                Mountpoint      FStype  Options         Dump    Pass#
/dev/gpt/efiboot0       /boot/efi       msdosfs rw              2       2
/dev/ada0p2             none            swap    sw              0       0
proc                    /proc           procfs  rw              0       0
fdescfs                 /dev/fd         fdescfs rw              0       0
```

**[/etc/rc.conf](https://github.com/KowalskiPi/TcBSD-CP6706-config/tree/main/etc)**

```properties
#fonts_setting----------
allscreens_flags="-f vgarom-thin-8x16"
#####################
zfs_enable="YES"
dumpdev="AUTO"
CXID_enable="YES"
MDPService_enable="YES"
ntpd_enable="YES"
pf_enable="YES"
TcSystemService_enable="YES"
runonce_enable="YES"
sendmail_enable="NO"
sendmail_submit_enable="NO"
sendmail_outbound_enable="NO"
sendmail_msp_queue_enable="NO"
dhcpcd_enable="YES"
#ifconfig_igb0="inet 192.168.137.137 netmask 255.255.255.0"
#defaultrouter="192.168.137.1"
#ifconfig_igb1="dhcp"
#dhcpcd_flags="--waitip --denyinterfaces igb0"
dhcpcd_flags="--waitip"
allscreens_kbdflags="-b quiet.off"
syslogd_flags="-ss"
devfs_system_ruleset="allow_usb_mount"
hostname="CP-5B3CCE"

#DAEMONS----------------------------
zfs_enable="YES"
#moused_enable="YES"
webcamd_enable="YES"
dbus_enable="YES"
kld_list="i915kms"
#gdm_enable="YES"
lightdm_enable="yes"

#RemoteDesktop----------------------
sshd_enable="yes"
xrdp_enable="yes"
xrdp_sessman_enable="yes"

#gateway_enable="true"

#TcHmi------------------------------
TcHmiSrv_enable="YES"
```

**[/etc/ttys](https://github.com/KowalskiPi/TcBSD-CP6706-config/tree/main/etc)**

```text
# name  getty                   type    status          comments
#
console none                    unknown off secure
#
ttyv0   "/usr/libexec/getty Pc"         xterm   onifexists secure
# Virtual terminals
# ....existing ttyv
ttyv8   "/usr/local/bin/xdm -nodaemon"  xterm   onifexists secure
```

## Enable TF2000 (HMI Server)

Install the TwinCAT HMI Server, configure the firewall (`pf`), and set up the database.

```shell
# Install and enable HMI service
$ doas pkg install TF2000-HMI-Server

# Unblock firewall temporarily to edit config
$ doas service pf stop

$ doas ee /etc/pf.conf
# >> pass in quick proto tcp to port 0000 keep state
# >> pass in quick proto tcp to port 1010 keep state
# >> pass in quick proto tcp to port 1020 keep state
# >> pass in quick proto tcp to port 8080 keep state

$ doas service pf start

# Enable HMI service
$ doas service TcHmiSrv start
```

> **Reminder:** Ensure `TcSystemService_enable="YES"` is added to `/etc/rc.conf` and your EFI boot partition is correctly mounted in `/etc/fstab`.

### SQLite Configuration

```shell
# Install and config SQLite server
$ doas pkg install sqlite

# Change directory to TcHmiProject
$ cd /usr/local/etc/TwinCAT/Functions/TF2000-Hmi-Server/service/TcHmiProject

# Change sqlite database as you like
$ doas sqlite storage.db
...
```

## Setup Kiosk Mode (Firefox)

We will configure Firefox to autostart in Kiosk mode, pointing to the local HMI server.

```shell
# Install firefox
$ doas pkg install firefox

# Edit autostart file
$ doas ee /home/SystemUser/.config/openbox/autostart.sh
```

Paste the following into `autostart.sh`:

```bash
#!/bin/sh

firefox --kiosk --private-window localhost:1010
xset dpms 0 0 0
xset s noblank
xset s off
```

Make the script executable:

```shell
$ chmod +x /home/SystemUser/.config/openbox/autostart.sh
```

## Repository & Resources

  * **GitHub Config Repo:** [https://github.com/KowalskiPi/TcBSD-CP6706-config](https://github.com/KowalskiPi/TcBSD-CP6706-config)
  * **Video Tutorial:** [TwinCAT/BSD on touch panel & TF2000 on TC/BSD](https://www.youtube.com/watch?v=u-EAIi6sR44)

---
