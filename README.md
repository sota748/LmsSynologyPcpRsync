# LmsSynologyPcpRsync - ReadMe
Glossary  
- <u>LMS (Lyrion Media Server)</u>: A music server that streams your entire local music library and online radio to multiple players throughout your home, (https://lyrion.org/)  
- <u>NAS</u>: A dedicated storage device connected to your network that lets multiple users store and access files from any computer. Synology NAS (https://www.synology.com/en-global)  
- <u>Docker</u>: A tool that packages an application and everything it needs into a single container, so it runs perfectly on any computer (and NAS)  
- <u>pCP (PiCorePlayer)</u>: A lightweight operating system that turns a Raspberry Pi into a high-quality music player for your Lyrion Music Server, (https://picoreplayer.org/)  
- <u>"OpenVPN"</u>: Software that creates a secure, encrypted tunnel to your home or office network from anywhere in the world.  
- <u>"samba"</u>: Software that connects Linux and Windows so they can easily share folders and printers.  
- <u>"rsync"</u>: A software tool that efficiently copies and synchronizes files between folders or computers by only moving the changes.  

## The motivation for the project:
I have a good friend who has Dementia at a young age (50 years old), and he loves music like me.  
He has problems operating, among other things, the CD player, so I have made him a pCP player, it is operated by him (with help from his wife).  

So that I can help from home, I have installed OpenVPN on it, this means that I from home can operate:  
- LMS webinterface (xxx.xxx.xxx.xxx:9000)  
- pCP webinterface, setup pages (xxx.xxx.xxx.xxx)  
- Files on the USB-HB, with Samba  
- SSH with PuTTY

But it was/is a hassle to keep the pCP USB-HD updated with the changes I made to home Music library on the NAS.  
So I decided to make an automatic update of the pCP USB-HD, so that it is always a mirror of my NAS library.  
- To achieve this I have used rsync.  

The whole setup have currently been running with 2-3 remote pCPs for over a year now without any problems, so I think it can be called a stable system.​

## System Description:
Principle diagram.  
![principlediagram](jpg/lms_synology_pcp_rsync_principle_diagram.jpg)

* **Home System:** Hardware / Software  
<u>NAS:</u> Synology DS723+ / DSM 7.2.1-69057 Update 5  
<u>Router:</u> TP-Link Archer AX50 v1.0 / 1.0.14 Build 20240108 rel.42655(4555)  
<u>LMS:</u> Docker on Synology / Image: lmscommunity/lyrionmusicserver:dev

* **Remote System:** Hardware / Software    
<u>LMS/Player:</u> Raspberry Pi 4B+ / piCorePlayer 10.0.0 – 64 bit  



# Open VPN (remote pCP contol)
## Home System - Setup:   
* **Synology NAS:** Rsync

* **Synology NAS:** Rsync