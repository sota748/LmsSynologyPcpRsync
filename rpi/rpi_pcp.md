[Return to README](../README.md)  

# pCP (PiCorePlayer) - Raspberry Pi   

## Install SD-Card:  
Follow the instructions in the link below (rigth click and "Open in a new Tap/Window")  
[Install pCP SD-Card](https://docs.picoreplayer.org/getting-started/)  
After "Step 3 - Boot piCorePlayer", follow the instructions below  

## pCP webinterface, setup pages:  
Link/IP: xxx.xxx.xxx.xxx  
pCP webinterface: Menu  
![pCP menu](rpi_pcp_1_menu.png)  

PiCorePlayer is used as both Player and Server  

**Setup General:**  
Start first time  
- pCP system password = xxxxxx (safe password, I use the same on all remote pCP's)  
- Do you want to check for updates? = Yes  
- Set Hostname = pcpX (X = number)  
- Enable SSH = Yes  
- Enable NTP = Leave Default  
- Select pCP Mode = Yes  
- Select pCP Theme = Light  
- pCP Repo = Next  
- Audio Detection = Setup later on Squeezelite Page (Save)  
- Resize SD card = 8000MB  

**Main Page:**  
- Install piCorePlayer extensions (Main Page -> Extensions -> main repository)  
- nano.tcz  
- openvpn.tcz (for OpenVPN)  
- rsync.tcz  (for rsync)  

**Squeezelite Settings:**  
- Audio output device settings (follow the instructions on the page)  

**Wifi Settings:**  
- Wifi to mobile hotspot (only used for testing)  

**Tweaks:**  
- Nothing  

**Drives:**  
- Mount USB Disk = /mnt/hd  
- Set USB Mount  
- Set Write Permissions  
- Install Samba for pCP  
- Server Name = pcpX (X = number)  
- Server WorkGroup = WORKGROUP  
- Share Name = hd  
- Share path = /mnt/hd  
- Create File Mode = 0775  
- User = tc, Password = xxxxxx (I use the same as, Setup General/piCorePlayer system password)  
- Windows file browser = \\pcpX\hd  

**Lyrion:**  
- Install LMS  
- Set Branch = Stable  
- Start LMS  
- Save LMS Server Cache and Preferences to Mounted Drive = USB Disk  
- Move LMS Data  
- BackUp + Reboot (Main Page)  

## LMS webinterface  
Link/IP: xxx.xxx.xxx.xxx:9000  
LMS webinterface: Menu  
![LMS menu](rpi_pcp_2_menu.jpg)  

### LMS Settings -> Server  
Start first time  

**Basic Settings**  
- Media Library Name: pcpX-lms  
- Set & Scan all Media Folders (7 pcs.)  
- Set Playlists Folder  

**Plugins**  
- Additional Browse Modes  
- Don't Stop The Music  
- Drag & drop music files to play  
- Find Artwork for Radio  
- Full text search  
- Material Skin  
- Music and Artist Information  
- Online Music Library  
- Podcasts  
- Presets Editor  
- Radio  
- Radio Now Playing  
- RadioNet  
- Random Mix  
- Remote Music Libraries  
- Report Analytics Data  
- Song Scanner  
- Sounds & Effects  
- Visual Statistics  

**Interface**  
- Time Format = hh:mm:ss (24h)  

**My Music**  
- Release types for Albums -> Try to recognize EPs and singles automatically when release type information is "album" or is missing = On  
- Release types for Albums -> Group an artist's lists by release type  

**Plugins -> Material Skin**  
- Use comment for disc title -> Contains only title  
- Show comment in info view -> On  

### LMS Settings -> Interface  
- Color -> For all players  
- Home screen items -> Scrollable lists = Explore + New Music + Randdom Releases + Recently Played + Radios  
- Home screen items -> Categories = My Music + Radio + Favorites + Apps + Extras  
- Home screen items -> Categories -> Browse modes = "Individual setup for each installation"  
- Defaults -> Save as default = "Save the first time when the setup is done the current installation"  

### LMS Settings -> Player  

**Exstra settings -> Audio**  
- Volume Control -> Output level is fixed at 100% (for best sound when not in use)  
