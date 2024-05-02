# 1. Hardware requirement 

## Minimal requirements
- Raspberry Pi (RPi) 4 2GB single board computers (SBC): 1 for the head node, plus as many cores as you want
- A multiport Netgear switch (as many ports as Rasberry Pis
- 10BaseT Cat6 ethernet cables (1 per Rasberry Pi)
- Power supplies for each Rasberry Pi (alternatively: use a PuE switch to power all Rasberry Pis)
- An 8GB flash drive for shared storage
- A 32GB SD card to boot the main node from
- Cooling device (e.g. USB desktop fan)
 
## Optional
- Example of casing:
  - 3D printed DIN Rail stand
  - 3D printed RPi cases

# 2. Setup: connecting together
- 

# 3. Initial configuration 
_TODO From https://github.com/carpentriesoffline/CarpentriesOffline.github.io/blob/main/rpiimage_step_by_step.md_

## Creating an SD card image: step-by-step

###  Setting up a Raspberry Pi

The official [Set up your SD card](https://projects.raspberrypi.org/en/projects/raspberry-pi-setting-up/2) is up to date as of 2/05/2024. The Raspberry Pi Imager work as expected on Windows.

* When using the The Raspberry Pi Imager,

![Screenshot 2024-05-02 120352](https://github.com/carpentriesoffline/CW24_Build_miniHPC/assets/1506457/c7d4fc94-0285-4d88-8a02-2d129480b4c3)

it will ask if the user wants to do any customisation

![Screenshot 2024-05-02 120407](https://github.com/carpentriesoffline/CW24_Build_miniHPC/assets/1506457/c0182ff7-6993-4f6d-9d25-6d637679e8fa)

User should select to apply some customisation to configure the Wi-Fi

![Screenshot 2024-05-02 120430](https://github.com/carpentriesoffline/CW24_Build_miniHPC/assets/1506457/16b0ad5e-cfcd-4762-9537-0e1d66a1d78f)

and enable SSH

![Screenshot 2024-05-02 120838 pixelise](https://github.com/carpentriesoffline/CW24_Build_miniHPC/assets/1506457/82195fe0-a26e-4d49-b393-674c5ea01bed)


### Setup for headless config (useful if you don't have a screen and keyboard to hand) -> TODO
* In the boot (small FAT32) partition on the SD card create an empty file called "ssh"
* If you're using WiFi to get access to the Pi, create a file called wpa_supplicant.conf in the boot partition. Paste in the following and set your network SSID and password appropriately.

```
#set this to your country code, gb=great britain
country=gb
update_config=1
ctrl_interface=/var/run/wpa_supplicant

network={
 scan_ssid=1
 ssid="my_networks_ssid"
 psk="my_networks_password"
}
```

#### Login to the Pi
Use SSH or login with a local console if you have a monitor attached.

#### Run the install script
* Login to your Raspberry Pi with an SSH client or on a local screen/keyboard and run the command:
* ```curl https://raw.githubusercontent.com/carpentriesoffline/carpentriesoffline-installer/main/setup.sh > setup.sh && bash ./setup.sh```

#### Change the password
* Run the `passwd` command. Leaving the default password will mean anybody in your workshop can login to your Pi and change settings on it.

#### Connect to Carpentries Offline
* After installing the Raspberry Pi will reboot.
* It will then switch the WiFi interface to access point mode and will be available as a network called carpentries-offline.
* Connect to the carpentries-offline WiFi network
* Visting [http://carpentriesoffline.org](http://carpentries.org) or [http://192.168.1.1](http://192.168.1.1)
* You should get links to the Carpentries Lessons and the Gitea server on the Raspberry Pi

#### Using PyPi and CRAN mirrors from your Pi
* These are downloaded to the Pi and placed in [http://192.168.1.1/pypi](http://192.168.1.1/pypi) and [http://192.168.1.1/miniCRAN](http://192.168.1.1/miniCRAN).
* You will need to update your settings to use these locations. (TODO: write instructions on how to do this)



# Installing SLURM/HPC

## Setting up the miniHPC login node
- Do an update and a full-upgrade:

```bash
sudo apt-get update
sudo apt-get full-upgrade
```

- Install the following packages:

```bash
sudo apt-get install -y nfs-kernel-server lmod ansible slurm munge nmap \ 
nfs-common net-tools build-essential htop net-tools screen vim python3-pip \
dnsmasq slurm-wlm
```


- Setup the Cluster network

Place the following into /etc/network/interfaces

```
auto eth0
allow-hotplug eth0
iface eth0 inet static
  address 192.168.5.101
  netmask 255.255.255.0
source /etc/network/interfaces.d/*
```  

- Modify the hostname

```bash
echo pixie001 | sudo tee -a /etc/hostname
```

- Configure dhcp by entering the following in the file `/etc/dhcpd.conf`

```bash
interface eth0
static ip_address=192.168.0.1/24
static routers=192.168.0.1
static domain_name_servers=192.168.0.1
```

- Configure dnsmasq by entering the following in the file `/etc/dnsmasq.conf`

```bash
interface=eth0
bind-dynamic
domain-needed
bogus-priv
dhcp-range=192.168.0.1,192.168.0.100,255.255.255.0,12h
```

- Configure shared drives by addeding the following at the end of the file `/etc/exports`

```bash
/sharedfs    192.168.0.0/24(rw,sync,no_root_squash,no_subtree_check)
/modules     192.168.0.0/24(rw,sync,no_root_squash,no_subtree_check)
```

- The `/etc/hosts` file should contain the following:

```bash
127.0.0.1	localhost
::1		localhost ip6-localhost ip6-loopback
ff02::1		ip6-allnodes
ff02::2		ip6-allrouters

127.0.1.1	pixie001

192.168.0.2	pixie002
192.168.0.3	pixie003
192.168.0.4	pixie004
192.168.0.5	pixie005
```


- Install ESSI
- 
```
mkdir essi
cd essi
wget https://raw.githubusercontent.com/EESSI/eessi-demo/main/scripts/install_cvmfs_eessi.sh
sudo bash ./install_cvmfs_eessi.sh
echo "source /cvmfs/software.eessi.io/versions/2023.06/init/bash" >> /etc/profile
```

- Create a shared directory

```bash
sudo mkdir /sharedfs
sudo chown nobody.nogroup -R /sharedfs
sudo chmod 777 -R /sharedfs
```


- Install a client node
Flash another SD card for a Raspberry Pi. Boot it up with internet access and run the following:

```sudo apt-get install -y slurmd slurm-client munge vim ntp ntpdate```

On a Linux laptop (or with a USB SD card reader) take an image of this:

```dd if=/dev/mmcblk0 of=node.img```

Copy node.img to the master Raspberry Pi's home directory.


- Setup PXE booting
Download the pxe-boot scripts:
```git clone https://github.com/carpentriesoffline/pxe-boot.git
cd pxe-boot
./pxe-install
```

Initalise a PXE node:
```
./pxe-add <serial number> ../node.img <IP address>  <node name>
```

for example:
```
./pxe-add fa917c3a ../node.img 192.168.5.105 pixie002
```

This will create an entry with the serial number in /pxe-boot and /pxe-root. 


