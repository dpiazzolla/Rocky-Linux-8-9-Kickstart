# Rocky Linux Kickstart


This repository contains the kickstart configuration files for Rocky Linux 8/9, specifically designed to create a kernel optimized for VFX and 3D post-production environments.

Usage
Download the your-kickstart.cfg file.
Use the file during the installation of Rocky Linux by following the official instructions.
Customizations

# 1. Network Interfaces Configuration
The kickstart file includes customizations for network interfaces during the post-installation phase. It performs the following actions:

Remove all existing network interface configurations:

rm -f /etc/sysconfig/network-scripts/ifcfg-*
Add configuration for interface enp4s0:


cat << EOF > /etc/sysconfig/network-scripts/ifcfg-enp4s0
TYPE=Ethernet
NAME=enp4s0
DEVICE=enp4s0
ONBOOT=yes
BOOTPROTO=dhcp
IPV6_AUTOCONF=yes
EOF
Add configuration for interface enp5s0:

cat << EOF > /etc/sysconfig/network-scripts/ifcfg-enp5s0
TYPE=Ethernet
NAME=enp5s0
DEVICE=enp5s0
ONBOOT=yes
BOOTPROTO=dhcp
IPV6_AUTOCONF=yes
EOF

# 2. SSH Key Configuration
The kickstart file sets up SSH access for the user fbfservice with a specific public key. This allows for secure and automated access to the system. The fbfservice user is an administrative account used by our team with the MAAS key to access the system once the deployment is complete.

cd ~fbfservice
mkdir .ssh
chmod 700 .ssh
chown fbfservice:frame .ssh
echo "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQC+..." >> .ssh/authorized_keys
chmod 600 .ssh/authorized_keys
chown fbfservice:frame .ssh/authorized_keys

# 3. Sudoers Configuration
The file also grants the fbfservice user sudo privileges without requiring a password, which is essential for certain administrative tasks.
echo "fbfservice ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers.d/fbfservice

# 4. System and User Configuration
The configuration includes setting the keyboard layout, system language, system timezone, and the root password (which must be updated with your own encrypted password). Additionally, it creates the frame group and centos and fbfservice users.

keyboard --vckeymap=it --xlayouts='it'
lang en_US.UTF-8
timezone Europe/Rome --utc
rootpw --iscrypted $6$xyzxyzxyzxyzxyzxyzxyzxyzxyzxyzxyzxyzxyzxyzxyzxyzxyzxyzxyzxyzxyzxyzxyzxyzxyzxyzxyzxyzxyz
group --name=frame --gid=1000
user --name=centos --gid=1000
user --name=fbfservice --gid=1000 --gecos="FBF Local Service"

5. Kdump Configuration
The kdump service is disabled to free up resources.


%addon com_redhat_kdump --disable --reserve-mb='auto'
%end

# 6. Password Policies
Sets password policies for root, user, and LUKS encryption.

%anaconda
pwpolicy root --minlen=6 --minquality=1 --notstrict --nochanges --notempty
pwpolicy user --minlen=6 --minquality=1 --notstrict --nochanges --emptyok
pwpolicy luks --minlen=6 --minquality=1 --notstrict --nochanges --notempty
%end

# 7. Reboot After Installation
The system will reboot automatically after the installation process is complete.
reboot

# 8. GRUB Configuration
After installation, it is recommended to modify the GRUB configuration file (/etc/default/grub) to optimize the system for VFX and 3D post-production. Here’s how you can do it:

sudo nano /etc/default/grub
Add or modify the following parameters to optimize performance:


GRUB_CMDLINE_LINUX="crashkernel=auto rhgb quiet intel_iommu=on iommu=pt"
Save the file and update GRUB:
sudo grub2-mkconfig -o /boot/grub2/grub.cfg

# 9. EFI Boot Configuration
You may also need to modify the GRUB configuration in the EFI boot directory (D:\EFI\BOOT\grub.cfg). Below is an example configuration that you can use:

set default="0"

function load_video {
  insmod efi_gop
  insmod efi_uga
  insmod video_bochs
  insmod video_cirrus
  insmod all_video
}

load_video
set gfxpayload=keep
insmod gzio
insmod part_gpt
insmod ext2

search --no-floppy --set=root -l 'ROCKY'

menuentry 'Install Rocky Linux 9.3 with Kickstart' --class fedora --class gnu-linux --class gnu --class os {
  linuxefi /images/pxeboot/vmlinuz inst.repo=hd:LABEL=ROCKY inst.ks=hd:LABEL=ROCKY:/kickstart.cfg quiet modprobe.blacklist=nouveau 
  initrdefi /images/pxeboot/initrd.img
}

menuentry 'Install Rocky Linux 9.3' --class fedora --class gnu-linux --class gnu --class os {
  linuxefi /images/pxeboot/vmlinuz inst.repo=hd:LABEL=ROCKY quiet
  initrdefi /images/pxeboot/initrd.img
}
menuentry 'Test this media & install Rocky Linux 9.3' --class fedora --class gnu-linux --class gnu --class os {
  linuxefi /images/pxeboot/vmlinuz inst.repo=hd:LABEL=ROCKY rd.live.check quiet
  initrdefi /images/pxeboot/initrd.img
}
submenu 'Troubleshooting -->' {
  menuentry 'Install Rocky Linux 9.3 in basic graphics mode' --class fedora --class gnu-linux --class gnu --class os {
    linuxefi /images/pxeboot/vmlinuz inst.repo=hd:LABEL=ROCKY nomodeset quiet
    initrdefi /images/pxeboot/initrd.img
  }
  menuentry 'Rescue a Rocky Linux system' --class fedora --class gnu-linux --class gnu --class os {
    linuxefi /images/pxeboot/vmlinuz inst.repo=hd:LABEL=ROCKY inst.rescue quiet
    initrdefi /images/pxeboot/initrd.img
  }
}

# Important Note

This kickstart configuration is tailored for creating a system optimized for VFX and 3D post-production. The customizations include specific network interface configurations, SSH key setup, user account configurations, and GRUB settings suitable for a production environment where automation and secure access are crucial.

Before using this kickstart file, make sure to:

Update the root password with a secure, encrypted password.
Review and modify network configurations as needed for your specific environment.
Ensure the SSH key setup is secure and the key used is your own.
Modify the GRUB configuration post-installation to suit your performance needs.

# Contributing

Contributions are welcome. Please open a pull request or issue for any suggestions or improvements.


