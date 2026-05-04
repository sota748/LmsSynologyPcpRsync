# LmsSynologyPcpRsync - ReadMe

## Description of the motivation for the project:
I have a good friend who has Dementia at a young age (50 years old), and he loves music like me.  
So I have made him a pCP player, it is operated by him (with help from his wife).  
So that I can help from home, I have installed OpenVPN on it, this means that I can operate pCP & Server interface USB-HD from home.

## System Description:
Principle diagram.  
![principlediagram](jpg/lms_synology_pcp_rsync_principle_diagram.jpg)

* **Home System:** Hardware / Software  
<u>NAS:</u> Synology DS723+ / DSM 7.2.1-69057 Update 5  
<u>Router:</u> TP-Link Archer AX50 v1.0 / 1.0.14 Build 20240108 rel.42655(4555)  
<u>LMS:</u> Docker on Synology / Image: lmscommunity/lyrionmusicserver:dev

* **Remote System:** Hardware / Software    
<u>LMS/Player:</u> Raspberry Pi 4B+ / piCorePlayer 10.0.0 – 64 bit  

## Home System - Setup:   
* **Synology NAS:** Rsync

* **Synology NAS:** VPN