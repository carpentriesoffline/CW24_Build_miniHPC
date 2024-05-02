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
