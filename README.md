# Rocky Linux Kickstart

This repository contains the kickstart configuration files for Rocky Linux 8/9, specifically designed to create a kernel optimized for VFX and 3D post-production environments.

## Usage

1. Download the `your-kickstart.cfg` file.
2. Use the file during the installation of Rocky Linux by following the [official instructions](https://rockylinux.org/).

## Customizations

### 1. Network Interfaces Configuration

The kickstart file includes customizations for network interfaces during the post-installation phase. It performs the following actions:

- **Remove all existing network interface configurations**:
 
  rm -f /etc/sysconfig/network-scripts/ifcfg-*

 cat << EOF > /etc/sysconfig/network-scripts/ifcfg-enp4s0
 TYPE=Ethernet
 NAME=enp4s0
 DEVICE=enp4s0
 ONBOOT=yes
 BOOTPROTO=dhcp
 IPV6_AUTOCONF=yes
 EOF

cat << EOF > /etc/sysconfig/network-scripts/ifcfg-enp5s0
TYPE=Ethernet
NAME=enp5s0
DEVICE=enp5s0
ONBOOT=yes
BOOTPROTO=dhcp
IPV6_AUTOCONF=yes
EOF

2. SSH Key Configuration
The kickstart file sets up SSH access for the user fbfservice with a specific public key. This allows for secure and automated access to the system.

SSH Key Setup:

cd ~fbfservice
mkdir .ssh
chmod 700 .ssh
chown fbfservice:frame .ssh
echo "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQC+..." >> .ssh/authorized_keys
chmod 600 .ssh/authorized_keys
chown fbfservice:frame .ssh/authorized_keys

3. Sudoers Configuration
The file also grants the fbfservice user sudo privileges without requiring a password, which is essential for certain administrative tasks.

Sudo Configuration:

echo "fbfservice ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers.d/fbfservice

4. System and User Configuration
Keyboard Layout:

keyboard --vckeymap=it --xlayouts='it'
System Language:

lang en_US.UTF-8
System Timezone:

timezone Europe/Rome --utc
Root Password:
This must be updated with your own encrypted password. The provided one is a placeholder:

rootpw --iscrypted $6$xyzxyzxyzxyzxyzxyzxyzxyzxyzxyzxyzxyzxyzxyzxyzxyzxyzxyzxyzxyzxyzxyzxyzxyzxyzxyzxyzxyzxyz
You can generate an encrypted password using the openssl passwd -6 command.
Group and Users:

group --name=frame --gid=1000
user --name=centos --gid=1000
user --name=fbfservice --gid=1000 --gecos="FBF Local Service"

5. Kdump Configuration
The kdump service is disabled to free up resources:


%addon com_redhat_kdump --disable --reserve-mb='auto'
%end

6. Password Policies
Sets password policies for root, user, and LUKS encryption:


%anaconda
pwpolicy root --minlen=6 --minquality=1 --notstrict --nochanges --notempty
pwpolicy user --minlen=6 --minquality=1 --notstrict --nochanges --emptyok
pwpolicy luks --minlen=6 --minquality=1 --notstrict --nochanges --notempty
%end
7. Reboot After Installation
The system will reboot automatically after the installation process is complete:

e
reboot
Important Note
This kickstart configuration is tailored for creating a system optimized for VFX and 3D post-production. The customizations include specific network interface configurations, SSH key setup, and user account configurations that are suitable for a production environment where automation and secure access are crucial.

Before using this kickstart file, make sure to:

Update the root password with a secure, encrypted password.
Review and modify network configurations as needed for your specific environment.
Ensure the SSH key setup is secure and the key used is your own.

Contributions are welcome. Please open a pull request or issue for any suggestions or improvements.



### Steps to Follow:

1. **Create a New Repository**:
   - Go to GitHub, click the "+" button in the upper right corner, and select "New repository".
   - Name the repository, for example, `rocky-linux-kickstart`.
   - Add a description, such as "Kickstart configuration files for Rocky Linux 8/9, optimized for VFX and 3D post-production".
   - Choose to make the repository public or private.
   - Add a README file.
   - Click "Create repository".

2. **Clone the Repository**:

   git clone https://github.com/your-username/rocky-linux-kickstart.git
   cd rocky-linux-kickstart
Add the Kickstart File:

Copy your kickstart file to the repository:

cp /path/to/your-kickstart.cfg .
Add the file to the repository:

git add your-kickstart.cfg
Commit the changes:

git commit -m "Add kickstart configuration for Rocky Linux 8/9"
Push the changes to GitHub:

git push origin main
Update the README File:

Edit the README.md to add usage instructions and details about the network interfaces customization and other configurations.
Push the README Updates:

git add README.md
git commit -m "Update README with usage instructions"
git push origin main
Following these steps, you will create and share your kickstart configuration for Rocky Linux on GitHub, making it accessible for others to use and contribute to. If you need further assistance, feel free to ask!






