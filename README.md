# XTLS-Iran-Reality
### Xray-core (V2ray) Server with Reality Protocol for bypassing internet censorship in Iran with TLS encryption.

If you need farsi translation check out this link https://telegra.ph/How-run-Reality-protocol-with-Xray-or-Sing-box-Core-with-iSegaro-04-18

If you want X-UI use this https://github.com/MHSanaei/3x-ui

And make sure you block Iranian IP's and domains in custom configuration.

- The main goal of this guide is to spread awereness on how to make one correctly.
- The configuration file [(config.json)](https://github.com/SasukeFreestyle/XTLS-Iran-Reality/blob/main/config.json) is the main key here that includes a correct CIDR-IP block so the server does not initiate a connection back to Iran as this is not "normal" behaviour for a (web)server.

****

### Notes
- This is a noob-friendly guide but if you are an experienced linux user you should make a new user without sudo-access to run xray and give right permissions to files.
- I wanted to make it easy for anyone non-technical to make a server without changing/creating users or editing permissions of files.
- I will also teach on how to use your Iranian IP for direct communication to Iranian websites/services without disconnecting the "VPN" using routing rules.
- <ins>This will not work with CDNs such as Cloudflare.</ins>
- This will only work with clients that use xray-core 1.8.0 or above. See [Clients](https://github.com/SasukeFreestyle/XTLS-Iran-Reality#clientapps-settings) for links to updated apps/programs.
- [V2rayNG](https://github.com/2dust/v2rayNG/releases) version 1.8.2+ or above for Android. 
- [FoXray](https://apps.apple.com/app/foxray/id6448898396) for Iphone.

****

This guide is written for Ubuntu 22.04 LTS but any Debian based distro should also work.

### What you need before starting this guide. Prerequisites
- VPS or any other computer / Virtual-Machine running Ubuntu 22.04 LTS or a Debian based distro.
- SSH or terminal/console access to your server.
- You need to know your username (the username when you log into Ubuntu)
- Port 443 open in your router or/and firewall.
- ### Optional (But not required)
- A Domain name, You can get a free domain name from https://freedns.afraid.org/ or https://www.noip.com/
- Domain name must be pointed to your IP hosting the server.


****
## First we need to do some kernel settings for performance and raise ulimits.

```
sudo nano /etc/sysctl.conf
```
Copy this at end of then file and save and close.
```console
net.ipv4.tcp_keepalive_time = 90
net.ipv4.ip_local_port_range = 1024 65535
net.ipv4.tcp_fastopen = 3
net.core.default_qdisc=fq
net.ipv4.tcp_congestion_control=bbr
fs.file-max = 65535000
```

Then run this command to edit limits.conf 
```
sudo nano /etc/security/limits.conf
```

Copy this at end of the file and save and close.
```console
* soft     nproc          655350
* hard     nproc          655350
* soft     nofile         655350
* hard     nofile         655350
root soft     nproc          655350
root hard     nproc          655350
root soft     nofile         655350
root hard     nofile         655350
```

Run this to apply settings.
```
sudo sysctl -p
```
## Install Xray (XTLS)

Create a folder called xray in your username home folder. You should be in this folder when you log in.

```
mkdir xray
```

Update Ubuntu package list and install unzip.
``` 
sudo apt-get update
```
```
sudo apt-get install unzip
```
Change directory to the newly created xray folder.

```
cd xray/
```
Download the latest geoasset file for blocking Iranian websites.
```
wget https://github.com/bootmortis/iran-hosted-domains/releases/latest/download/iran.dat
```

Download the latest version of XTLS-Xray-Core.

Link to release page.

https://github.com/XTLS/Xray-core/releases

To download the Xray-linux-64.zip file, we can use the wget command.
Then we will unzip the file.

```
wget https://github.com/XTLS/Xray-core/releases/download/v1.8.3/Xray-linux-64.zip
```
```
unzip Xray-linux-64.zip
```
Remove the Xray-linux-64.zip for easier future updates. See [updates](https://github.com/SasukeFreestyle/XTLS-Iran-Reality#how-to-update-to-latest-version)
```
rm Xray-linux-64.zip
```
Generate UUID for config.json save this for later.
Replace Secret with any random text/string
```
./xray uuid -i Secret
```
It should look something like this.
```console
92c96807-e627-5328-8d85-XXXXXXXXX
```

Generate Private and Public keys and save it for later
```
./xray x25519
```
It should look something like this.
```console
Private key: qBvFzkSMcgrXXXXXJu2VSt3-0dCy-XX8IXXXXXXXXXX
Public key: rhrL9r_VGMWtwXXXXHO_eAi5e4CIn_XXXXXXXXXXXXX
```
- The Private key is only for the server.
- The Public key is for your apps/clients.


Run this command to generate short IDs, You can have multiple short IDs or just one.
Save it for later.

```
openssl rand -hex 8
```
It should look something like this.
```console
d82fb387XXXXXXXX
```
## Install xray to boot at startup (Systemd-Service) create file or copy paste [xray.service](https://github.com/SasukeFreestyle/XTLS-Iran-Reality/blob/main/xray.service) file from this repository
Create service file.
```
sudo nano /etc/systemd/system/xray.service
```

```console
[Unit]
Description=XTLS Xray-Core a VMESS/VLESS Server
After=network.target nss-lookup.target
[Service]
# Change to your username <---
User=USERNAME
Group=USERNAME
CapabilityBoundingSet=CAP_NET_ADMIN CAP_NET_BIND_SERVICE
AmbientCapabilities=CAP_NET_ADMIN CAP_NET_BIND_SERVICE
NoNewPrivileges=true
#                       --->  Change to your username  <---
ExecStart=/home/USERNAME/xray/xray run -config /home/USERNAME/xray/config.json
Restart=on-failure
RestartPreventExitStatus=23
StandardOutput=journal
LimitNPROC=100000
LimitNOFILE=1000000
[Install]
WantedBy=multi-user.target
```
Remember to edit this file to your own ***USERNAME!***
The parts to edit are.
```console
User=USERNAME
Group=USERNAME
ExecStart=/home/USERNAME/xray/xray run -config /home/USERNAME/xray/config.json
```

Example
```console
User=SasukeFreestyle
Group=SasukeFreestyle
ExecStart=/home/SasukeFreestyle/xray/xray run -config /home/SasukeFreestyle/xray/config.json
```


Reload services and enable auto-start.
```
sudo systemctl daemon-reload && sudo systemctl enable xray
```

## Xray Configuration

Create a new file called config.json inside xray folder.
Copy contents of [config.json](https://github.com/SasukeFreestyle/XTLS-Iran-Reality/blob/main/config.json) from this repository to the file.
```
nano /home/USERNAME/xray/config.json
```

- Enter your UUID inside "YOUR UUID HERE"
- Enter your Private Key inside "PRIVATE KEY HERE" 
- Enter your Short ID inside "shortIds":
***
- Enter a destination ( "dest": ) and serverNames ("serverNames":).
- This must be a website outside of Iran that you can access with a regular internet connection.
- Destination website must have/support HTTPS/TLSv1.3 and H2.
- This config is preconfigured with www.google-analytics.com website but here is a list of other websites that I think works.
***
- www.samsung.com:443
- www.googletagmanager.com:443
- www.asus.com:443
- www.amd.com:443
- www.cisco.com:443
- www.linksys.com:443
- www.nvidia.com:443
- e.t.c


The parts to edit are.
```json
      {
         "listen":"0.0.0.0",
         "port":443,
         "protocol":"vless",
         "settings":{
            "clients":[
               {
                  "id":"UUID HERE", // Your generated UUID here.
                  "flow":"xtls-rprx-vision"
               }
            ],
            "decryption":"none"
         },
         "streamSettings":{
            "network":"tcp",
            "security":"reality",
            "realitySettings":{
               "show":false,
               "dest":"www.google-analytics.com:443", // Edit to a website/server that works without VPN outside of Iran
               "xver":0,
               "serverNames":[
                  "www.google-analytics.com" // (SNI) Same as "dest" but without portnumber.
               ],
               "privateKey":"PRIVATE KEY HERE", // Private key you generated earlier.
               "minClientVer":"1.8.0",
               "maxClientVer":"",
               "maxTimeDiff":0,
               "shortIds":["SHORT ID HERE" // Short ID
               ]
            }
         },
```
Example
```json
"id":"92c96807-e627-5328-8d85-XXXXXXXXX",
"privateKey":"qBvFzkSMcgrXXXXXJu2VSt3-0dCy-XX8IXXXXXXXXXX",
"shortIds":["d82fb387XXXXXXXX"]
```

Now start xray and check if xray is running it should now say Active: active (running).
```  
sudo systemctl start xray && sudo systemctl status xray
``` 
```console
● xray.service - XTLS Xray-Core a VMESS/VLESS Server
     Loaded: loaded (/etc/systemd/system/xray.service; enabled; vendor preset: enabled)
     Active: active (running) since Tue 2023-02-14 18:31:07 CET; 22min ago
   Main PID: 338362 (xray)
      Tasks: 16 (limit: 9365)
     Memory: 279.6M
        CPU: 5min 28.315s
``` 
Done! Now test the server with your clients.

## Client/Apps (Settings)


### V2rayNG (Android)
In V2rayNG press + then pick "Type manually[VLESS]"

Settings also apply to V2rayN (Windows).

Remember to set (uTLS) Fingerprint to Chrome.

- Remarks/Alias
  -  Name of the server, choose whatever name you want.
- Address
  - IP or domain name of your server.
- Port: 443
- id:
  - Your UUID in config.json
- Flow: xtls-rprx-vision
- Encryption: None
- Network: TCP
- TLS: Reality
- SNI: www.google-analytics.com (serverNames)
- uTLS/Fingerprint: randomized or Chrome (I prefer randomized).
- PublicKey: rhrL9r_VGMWtwXXXXHO_eAi5e4CIn_XXXXXXXXXXXXX, The public (not private) key you generated earlier.
- ShortId: d82fb387XXXXXXXX , The short ID you generated earlier.

![photo_2023-04-14_16-33-50](https://user-images.githubusercontent.com/2391403/233698254-7999d127-6adc-4170-847b-df796b692ff1.jpg)


If you want to be able to visit Iranians websites without disconnecting the VPN follow the instructions in the video below.

This will also make it harder for government to see that you are using a VPN.

- Connect to your server then go to Settings -> Geo asset files -> press download cloud to download new Geo asset files.

- Go to Settings -> Custom Rules -> Direct URL or IP.

Enter
```
geoip:private,
geosite:private,
geoip:ir,
geosite:category-ir
```

- Then save and reconnect to your server. Try browsing to youtube and tci.ir both will now work at the same time.

Video Instructions:


https://user-images.githubusercontent.com/2391403/235455406-96746fe5-fa45-43de-9c2a-9e9cca51f10d.mp4




***

### V2rayN (Windows)
[V2rayN 6.21+](https://github.com/2dust/v2rayN/releases)

![v2rayN](https://user-images.githubusercontent.com/2391403/233697274-4dc688a3-be79-4015-ac0d-4d7646eaa96e.PNG)

***

### Nekoray (Windows/Linux)
[Nekoray 2.25+](https://github.com/MatsuriDayo/nekoray/releases)

Change core to sing-box in "Basic Settings".

![nekoraysing](https://user-images.githubusercontent.com/2391403/232086925-16ede576-bc5f-4200-b3f1-a0f6dc6ef319.PNG)

![nekosettings](https://user-images.githubusercontent.com/2391403/233697238-e8f016f0-4971-45a3-8dfe-1e79f5d8ba84.PNG)


***

### Iphone/Mac
[FoXray](https://apps.apple.com/app/foxray/id6448898396)

Pictures/Screenshots comming soon.

***

## Routing rules
For routing rules for each client see bootmortis excellent guides. 
https://github.com/bootmortis/iran-hosted-domains

Link to some other routing rules for [V2rayNG](https://github.com/SasukeFreestyle/XTLS-Iran-Reality/issues/9) and [FoXray](https://github.com/SasukeFreestyle/XTLS-Iran-Reality/issues/10)

***

## How to update to latest version
If a new version of Xray is published and you want to update to the latest version do this easy steps.

- Log into your machine with SSH.

Change directory to your xray folder.
```
cd xray/
```
wget the latest release.
```
wget https://github.com/XTLS/Xray-core/releases/download/v1.8.3/Xray-linux-64.zip
```

This command will stop the xray service and remove old files and start xray service again.
```
sudo systemctl stop xray && rm geo* && rm LICENSE && rm README.md && rm xray && unzip Xray-linux-64.zip && sudo systemctl start xray
```
Make sure xray is running by entering this command.
```
sudo systemctl status xray
```
Remove the zipfile.
```
rm Xray-linux-64.zip
```
Done!

## Credits
XTLS-core Team / v2fly

[@bootmortis](https://www.github.com/bootmortis) for Iranian domain list and routing rules.

And many others.
