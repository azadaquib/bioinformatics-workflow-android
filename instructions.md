# Installing Ubuntu on rooted Android device using Termux

This guide is based on [LinuxDroidMaster's guide](https://github.com/LinuxDroidMaster/Termux-Desktops/blob/main/Documentation/chroot/ubuntu_chroot.md) but adds a few things for installing BwB application

# Assumptions

- You have a rooted device
- You have magisk installed
- You have installed the [busybox module](https://github.com/Magisk-Modules-Repo/busybox-ndk) 
- You have installed [termux](https://github.com/termux/termux-app) app on your device

# Steps to install Ubuntu

## Start termux and install the following packages

```
pkg update \
&& pkg install x11-repo root-repo \
&& pkg install termux-x11-nightly sudo pulseaudio wget
```

## Start root shell on Termux

The su command has some issues in termux. Rather than that I found `sudo su` works much better

```
sudo su
```

Your prompt should change from `~ $` to `# `

# Install Ubuntu 22.04 base files

Create a directory for chroot at `/data/local`

```
mkdir -p /data/local/tmp/ubuntu
chmod 755 /data/local/tmp/ubuntu
chown root:root /data/local/tmp/ubuntu
cd /data/local/tmp/ubuntu
```

Download the Ubuntu distro

Refer the suitable release here. Download an arm64 build
https://cdimage.ubuntu.com/ubuntu-base/releases/

```
curl https://cdimage.ubuntu.com/ubuntu-base/releases/24.04.4/release/ubuntu-base-24.04.4-base-amd64.tar.gz -o ubuntu.tar.gz

```

Extract the downloaded file

```
tar xpvf ubuntu.tar.gz --numeric-owner
mkdir -p sdcard
```

Go back to the `tmp` directory and create the `startu.sh` script

```
cd ..
busybox vi startu.sh
```

Copy the `startu.sh` content from the repository and make it executable

```
chmod +x startu.sh
sh startu.sh
```

If successful the prompt should change to `root@localhost`

We will setup network and users with the following commands

```
echo "nameserver 8.8.8.8" > /etc/resolv.conf
echo "127.0.0.1 localhost
::1 localhost" > /etc/hosts

groupadd -g 3003 aid_inet
groupadd -g 3004 aid_net_raw
groupadd -g 1003 aid_graphics
usermod -g 3003 -G 3003,3004 -a _apt
usermod -G 3003 -a root

apt update && apt upgrade

apt install nano vim net-tools sudo git
```

Timezone should automatically be configured. If not run

```
dpkg-reconfigure tzdata
```

Create a user called `ubuntu` or whatever you like

```
groupadd storage
groupadd wheel
useradd -m -g users -G wheel,audio,video,storage,aid_inet -s /bin/bash ubuntu
passwd ubuntu
```

Add the created user to sudoers file to have superuser privileges: 
```
visudo
```
Add this line (replace `ubuntu` with your prefered name): 
```
ubuntu ALL=(ALL:ALL) ALL
```

Switch to the created user: 
```
su - ubunut
```
And generate locales: 
```
sudo apt install locales
sudo locale-gen en_US.UTF-8
```

Install Desktop Environment: 

```
sudo apt install xubuntu-desktop
```

Run `exit` till you retrun to termux `~ $` shell

Reboot device

## Start Ubuntu

Start termux and go to root user

```
sudo su
```

Go to `tmp` directory

```
cd /data/locat/tmp`
```

Run the startu.sh command

```
sh startu.sh
```

You can uncomment line 24 and comment line 21 to start Ubuntu in a graphics environment. Ensure that you have installed termux-x11




