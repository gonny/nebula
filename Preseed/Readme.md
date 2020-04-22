# Unattended deployment
This installation is **net** install. The machine must have internet connection. In the best case the machine must obtain network information from DHCP.
Main motivation is to create automated test environment for ansible. This script shall deploy Proxmox into Proxmox or other virtualisation platform.

We should be able to test different user and deployment settings.

Download boot image and extract it. Then we may use it as an boot option during VM deployment
## Debian 
```bash
cd /tmp
wget http://ftp.debian.org/debian/dists/stretch/main/installer-amd64/current/images/netboot/netboot.tar.gz
tar xzf netboot.tar.gz
mv debian-installer/amd64/linux debian-installer/amd64/initrd.gz .
rm -rf debian-installer netboot.tar.gz ldlinux.c32 pxelinux.0 pxelinux.cfg version.info
```

## Ubuntu 18 LTS
```bash
cd /tmp
wget http://archive.ubuntu.com/ubuntu/dists/bionic-updates/main/installer-amd64/current/images/netboot/netboot.tar.gz
tar xzf netboot.tar.gz
mv ubuntu-installer/amd64/linux ubuntu-installer/amd64/initrd.gz .
rm -rf ubuntu-installer netboot.tar.gz ldlinux.c32 pxelinux.0 pxelinux.cfg version.info
```

Create VM in Proxmox
* The --https://www.codepile.net/raw/v5RPEwZ8.ini-- is link to the preseed file. It can be also USB file or some ftp/tftp.
* Following configuration is for Jack (at) office
```bash
export VM=$(pvesh get /cluster/nextid | grep -oE '[0-9]*')

qm create $VM \
--scsi0 motv:8 \
--net0 virtio,bridge=vmbr0 \
--bootdisk scsi0 \
--ostype l26 \
--keyboard en-us \
--scsihw virtio-scsi-pci \
--memory 1024 \
--balloon 4096 \
--cores 4 \
--args '-kernel /tmp/linux -initrd /tmp/initrd.gz -append "preseed/url=https://www.codepile.net/raw/v5RPEwZ8.ini debian-installer/allow_unauthenticated_ssl=true debian-installer/locale=en_US debian/priority=critical vga=normal debian-installer/keymap=uk console-keymaps-at/keymap=uk console-setup/layoutcode=en_US keyboard-configuration/xkb-keymap=us netcfg/choose_interface=auto localechooser/translation/warn-light=true localechooser/translation/warn-severe=true console-setup/ask_detect=false netcfg/get_hostname=PRESEED netcfg/get_domain=local FRONTEND_BACKGROUND=original"'

Ubuntu - https://www.codepile.net/raw/xG9nMw4Z.ini
Debian - https://www.codepile.net/raw/v5RPEwZ8.ini

qm start $VM
# Check status of the installation and remove args after powerOff the VM
qm start $VM \
&& ((while (qm status $VM | grep -o running); do sleep 10; done) && qm set $VM -delete args) \
&& qm start $VM
```

Manipulation with QM
```
qm stop $VM; qm destroy $VM
```


## Download boot
# Unattended deployment information sources
* [https://www.debian.org/releases/jessie/amd64/apbs02.html.en] (Information about Debian preseed function)
* [https://gitlab.com/morph027/pve-infra-poc/-/tree/master] (PVE IaaS using ansible - PoC)
* [https://pve.proxmox.com/wiki/Install_Proxmox_VE_on_Debian_Stretch] Proxmox VE on Debian Stretch installation
* [https://morph027.gitlab.io/post/pve-kickseed/] (A bit old example project using manual args)
