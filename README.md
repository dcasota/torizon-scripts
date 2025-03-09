# torizon-scripts

In Switzerland, a company called Toradex created an open source platform Torizon OS that simplifies the development and maintenance of embedded Linux. It is claimed to be ready to use on devices that require high reliability, so you can develop your application without worrying about the operating system and its maintenance.

![image](https://github.com/user-attachments/assets/3d71052a-9aa0-4260-b3e5-244b4d2f4af8)

Toradex focuses on embedded solutions for the industry. They develop their own carrier boards. Computer on Modules (CoM) / System on Modules (SoM) offer reliable and cost-effective embedded platform for building end-products. Toradex offers an extensive range of CoMs using leading Arm based System on Chips (SoC). See supported hardware on https://developer.toradex.com/hardware.
SoM family: Aquila, Verdin, Apalis, Colibri
Accessoires: Display, Wireless, Camera, etc. 

![image](https://github.com/user-attachments/assets/d25a3bdc-9052-453e-97c3-7e353c84161f)

From a software perspective, Linux has been chosen as the preferred platform, see  
- https://www.toradex.com/de/operating-systems  
- https://developer.toradex.com/torizon/  
- https://www.torizon.io/open-source-community  
- https://github.com/commontorizon/Documentation  

## Jumpstart to Torizon Development Environment
Let's start with Torizon Development Environment. What is supported? See Torizon Features https://www.torizon.io/open-source-community#torizon-feature

![image](https://github.com/user-attachments/assets/4cb82419-c8d9-45d2-829c-b9a982c046c7)

### x86_64 virtual machine builder system
Let's start with setup recipe for x86_64 https://github.com/torizon/meta-toradex-torizon/blob/scarthgap-7.x.y/docs/README-x86.md.

The following prerequisite has been tested even it's not directly supported by Toradex.  
- Microsoft x86_64 Windows 11 24H2
- Broadcom Workstation 17.6.2
- Virtual machine vhw21 for VMware Photon OS 64bit, 8x2 vcpu, 32GB ram, 200GB disk space, setup using photon-5.0-21b41f540.x86_64.iso (or 5.0 GA)

```
tdnf install -y build-essential tar wget python3 python3-curses chrpath git libxcrypt libxcrypt-devel

SETUP_USER="youruser"
useradd $SETUP_USER -m -g users -G sudo,wheel
chage -m 0 $SETUP_USER
echo "$SETUP_USER ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers

su -l $SETUP_USER

mkdir ~/bin
PATH=~/bin:$PATH
curl http://commondatastorage.googleapis.com/git-repo-downloads/repo > ~/bin/repo

wget https://github.com/openembedded/bitbake/archive/refs/tags/2.10.3.tar.gz
tar -xzvf 2.10.3.tar.gz
cd bitbake-2.10.3/
export PATH=$(pwd)/bin:$PATH
cd ..
sudo rm 2.10.3.tar.gz


git config --global user.email "setupuser@example.com"
git config --global user.name "setupuser"

mkdir common-torizon; cd common-torizon
mkdir layers; cd layers
git clone git://git.yoctoproject.org/poky -b scarthgap
git clone git://git.yoctoproject.org/meta-intel -b scarthgap
git clone https://github.com/torizon/meta-toradex-torizon.git -b scarthgap-7.x.y
git clone https://github.com/uptane/meta-updater.git -b scarthgap
git clone https://git.yoctoproject.org/meta-virtualization -b scarthgap
git clone https://github.com/openembedded/meta-openembedded -b scarthgap
cd ..
ln -s layers/meta-toradex-torizon/scripts/setup-environment setup-environment

wget https://linuxsoft.cern.ch/cern/centos/7/updates/x86_64/Packages/Packages/diffstat-1.57-4.el7.x86_64.rpm
sudo rpm -ivh diffstat-1.57-4.el7.x86_64.rpm
sudo rm diffstat-1.57-4.el7.x86_64.rpm

wget https://yum.oracle.com/repo/OracleLinux/OL8/codeready/builder/x86_64/getPackage/rpcgen-1.3.1-4.el8.x86_64.rpm
sudo rpm -ivh rpcgen-1.3.1-4.el8.x86_64.rpm
sudo rm rpcgen-1.3.1-4.el8.x86_64.rpm

# review
wget https://git.altlinux.org/tasks/332501/build/100/x86_64/rpms/libzstd-1.5.5-alt2.x86_64.rpm
wget https://git.altlinux.org/tasks/332501/build/100/x86_64/rpms/zstd-1.5.5-alt2.x86_64.rpm
wget https://git.altlinux.org/tasks/332501/build/100/x86_64/rpms/pzstd-1.5.5-alt2.x86_64.rpm
sudo rpm -ivh libzstd-1.5.5-alt2.x86_64.rpm --force
sudo rpm -ivh zstd-1.5.5-alt2.x86_64.rpm --force
sudo rpm -ivh pzstd-1.5.5-alt2.x86_64.rpm --force
sudo rm libzstd-1.5.5-alt2.x86_64.rpm
sudo rm zstd-1.5.5-alt2.x86_64.rpm
sudo rm pzstd-1.5.5-alt2.x86_64.rpm


MACHINE=intel-corei7-64 . setup-environment build-corei7-64

umask 022
bitbake torizon-minimal
```

Hint: If the build process stops, check and defect the issues and restart `Bitbake torizon-minimal`.

With recipe above you get the first torizon image. 

