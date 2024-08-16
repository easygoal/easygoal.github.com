---
layout: post
category: guideline
tagline: ""
keywords: server
tags: ["guideline"]
header:
lang: zh_CN 
date: 2019-02-11
title: Build a PXE boot server environment
---

This is an outline of how to build a PXE (**P**reboot e**X**ecution **E**nvironment) server to boot one or many computers over the network. This might be helpful if you don’t have a CD/DVD drive in the computer you want to do a clean install of the OS on, or if you want to build many identical computers at once. Building your PXE server as a virtual machine is highly recommended, as it becomes very portable and easy to deploy.

On the PXE client(s) (servers or computers) on which you want to install the OS, make sure that BIOS is configured to use the PXE Boot option at the top of the list of boot devices.

For this example I will use the following:
```
PXE server IP:   192.168.1.100
TFTP directory:  /export/pxe
ISO repository:  /export/iso
OS repository:   /export/os (followed by distribution/version/type/architecture)

Example: /export/os/Ubuntu/12.04/server/i386
```
Change the above values as needed for your installation.

For the sake of simplicity and to have a Web UI for some tasks I will be using ![OpenMediaVault](http://openmediavault.org/), or OMV, (based on Debian Squeeze – 6.x) as the PXE/TFTP/NFS server. You will also need a DHCP server on the LAN. If you don’t have a DHCP server, I recommend that you install an OMV plugin to provide those services on your TFTP/PXE server. Alternatively, you could use the DHCP function on a firewall. In the past I’ve successfully used pfSense as the DHCP server with the advanced settings ‘TFTP server’ and ‘Enable network booting’ options turned on.

## 1. Configure OpenMediaVault
### Create shares
Under **Access Rights Management > Shared Folders**, create the following shares:

- PXE – for TFTP/PXE configuration files.
- ISO – for ISO files to be extracted into the OS directory.
- OS – for operating system files.

### Enable the NFS server
Under **Services > NFS**, on the ‘Settings’ tab, check the **Enable** box.

### Configure NFS shares
On the ‘Shares’ tab, create the the following NFS shares:

- PXE – for TFTP/PXE configuration files.
- ISO – for ISO files to be extracted into the OS directory.
- OS – for operating system files.

## 2. Configure the TFTP server
In the OMV UI, go to Services > TFTP and make sure Enable is checked. For Shared folder, make sure to select pxe.

### Install SYSLINUX
You’ll need SYSLINUX to run your PXE server. If you haven’t already done so, under **Services > SSH**, make sure to enable SSH access to OpenMediaVault. Log in via SSH as root and run:
```
apt-get install syslinux
```
… and copy the pxelinux.0 file into place:
```
cp /usr/lib/syslinux/pxelinux.0 /export/pxe
```
## 3. Configure OpenMediaVault to act as DHCP server
Having a DHCP server is crucial for your PXE boot environment to work. Chances are you already have a DHCP server, and if you do, all you need to do is to make sure it’s configured to serve up the TFTP boot file, in this case the pxelinux.0 file. That said, if you don’t have a DHCP server, using the **openmediavault-dnsmasq** plugin is very handy. Simply download and install it as you would with any other OMV plugin. Once it’s installed, go to **Services > Local DNS / DHCP** and on the Settings tab:

Under ‘General’:

- Check Enable.
- Provide a Domain name.

Under ‘DHCP Settings’:

- Check Enable.
- Fill out the Gateway field.
- Fill out the First IP address field.
- Fill out the Last IP address field.
- For the TFTP Boot File, enter pxelinux.0.

When it’s all filled out, click OK at the bottom of the page.

## 4. Upload ISO files
Upload the ISOs to the /export/iso/ directory we just created. You can upload them any way you want, but using scp is probably the fastest if you know how to use it. However, any SFTP client, like Cyberduck, would work too.

## 5. Create the Ubuntu PXE Environment
In addition to the pxelinux.cfg and ISO directories you will also need a directory for each operating system you want to provide an installer for. In this example I’m using Ubuntu 12.04 Server with 32-bit (i386) and 64-bit (amd64) options.

```
mkdir -p /export/pxe/pxelinux.cfg
mkdir -p /export/pxe/Ubuntu/12.04/server/i386
mkdir -p /export/pxe/Ubuntu/12.04/server/amd64
mkdir -p /export/os/Ubuntu/12.04/server/i386
mkdir -p /export/os/Ubuntu/12.04/server/amd64
```

## 6. Mount and copy the ISO contents

Now it’s time to mount the ISOs and copy the contents to the OS directories we just created. Below is an example of extracting the 64-bit version of Ubuntu 12.04 Server. Adjust the commands based on the distribution you want to extract from an ISO.

Temporarily mount the ISO:

```
mount -o loop /export/iso/ubuntu-12.04-server-amd64.iso /mnt/loop
```

Copy vmlinuz to PXE export:

```
cp /mnt/loop/install/vmlinuz /export/pxe/Ubuntu/12.04/server/amd64
```

Copy initrd.gz to PXE export:

```
cp /mnt/loop/install/netboot/ubuntu-installer/amd64/initrd.gz /export/pxe/Ubuntu/12.04/server/amd64/
```

Copy contents of ISO to OS export:

```
cp -Ra /mnt/loop/* /export/os/Ubuntu/12.04/server/amd64/
```

Unmount the ISO:
```
umount /mnt/loop/
```

## 7. Create a PXE menu to select OS from

```
cp /usr/lib/syslinux/vesamenu.c32 /export/pxe
```

## 8. Create an OS menu config file

```
mkdir /export/pxe/Ubuntu
nano /export/pxe/Ubuntu/Ubuntu.menu
```

```
LABEL 1
	MENU LABEL Ubuntu 12.04 Server (32-bit)
	KERNEL Ubuntu/12.04/server/i386/vmlinuz
	APPEND boot=casper netboot=nfs nfsroot=192.168.1.100:/export/os/Ubuntu/12.04/server/i386/install initrd=Ubuntu/12.04/server/i386/initrd.gz
	TEXT HELP
	Boot the Ubuntu 12.04 Server 32-bit CD
	ENDTEXT
LABEL 2
	MENU LABEL Ubuntu 12.04 Server (64-bit)
	KERNEL Ubuntu/12.04/server/amd64/vmlinuz
	APPEND boot=casper netboot=nfs nfsroot=192.168.1.100:/export/os/Ubuntu/12.04/server/amd64 initrd=Ubuntu/12.04/server/amd64/initrd.gz
	TEXT HELP
	Boot the Ubuntu 12.04 Server 64-bit CD
	ENDTEXT
```
## 9. Other useful OpenMediaVault plugins

### Website

This is useful for serving up installation files via HTTP. If you decide to serve your files over HTTP, install and enable the openmediavault-website plugin. The simplest way option is probably to run it on alternate port 8181, since OMV already runs on port 80. Then, point the “website” to the OS share. Because you may want to be able to browse the OSes you’ve uploaded, under ‘Options’ section, you might want to check the option for: “If a URL which maps to a directory is requested, and there is no DirectoryIndex (e.g., index.html) in that directory, then a formatted listing of the directory will be returned.”

## 10. Boot the computer(s)

Boot the computer(s) you want to install a new OS on. Again, you may need to update the BIOS settings on the computers you want to PXE boot to enable PXE boot on them. Once it PXE boots you should be presented with the boot menu. Simply select your preferred OS to install to begin the installation.

[Original Link](https://martinlanner.com/2012/07/28/build-a-pxe-boot-server-environment/)

