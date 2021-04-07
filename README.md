# Kali Linux setup and config

## Download Kali for Raspberry (my model is PI 4 Model B with 8GB)

Site:       https://www.offensive-security.com/kali-linux-arm-images/  
Location:   https://images.kali.org/arm-images/kali-linux-2021.1-rpi4-nexmon-64.img.xz  
Filename:   kali-linux-2021.1-rpi4-nexmon-64.img.xz  
SHA256Sum:  f5e126f33d32882f526e16b5148bd8b84a4e7c351bdd0eb9cfe3da2176580181  

## Flash SD Micro Card (my model is SanDisk 128GB) using Etcher

https://www.balena.io/etcher/  

## First login 

Username:   kali  
Password:   kali

## Finalize setup

### Change keyboard layout
Open Terminal:  

either use:  

```bash
sudo dpkg-reconfigure keyboard-configuration 
```
or directly edit the keyboard config file:  
[Reference can be found here](/xkb-map-file.txt)
```
sudo nano /etc/default/keyboard
```

My recommendation for Swiss German:

```console
XKBMODEL="pc105"  
XKBLAYOUT="ch"  
XKBVARIANT="de_nodeadkeys"  
XKBOPTIONS="terminate:ctrl_alt_bksp"  

BACKSPACE="guess"
```
For the changes to take effect:  

```bash
sodo service keyboard-setup restart  
```


### Change password
### Update installation
Open Terminal:  
```bash
sudo apt update && sudo apt dist-upgrade -y
```

Configuring console-setup wizard -> Latin2 - central Europe and Romanian  
Configuring wireshark-common -> No 