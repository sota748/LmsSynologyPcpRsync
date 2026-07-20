# rsync - NAS
[Return to README](../README.md)

## Gathering knowledge:
To understand how rsync works on a Synology NAS, I have found good information at the links below. (rigth click and "Open in a new Tap/Window") 
- **Synology forum:** How to enable SSH key authentication on Synology NAS, https://community.synology.com/enu/forum/1/post/136213  
- **YouTube:** Enable Key Based SSH Authentication For Synology Servers, https://www.youtube.com/watch?v=XN9SuzV08Ew  
- **YouTube:** Use a RaspberryPi as an Offsite / Remote Backup for your Critical Data!, https://www.youtube.com/watch?v=ffx4HhNLqfI  

## Step 1: NAS rsync User (DSM 7.x):
I have created a new user "rsync" on the NAS which is only used for rsync.  
The "rsync" user belongs to the Admin group (only Admins can SSH)  
Give the "rsync" user the desired Permissions  
  
* **Step 1a: Create NAS user (Control Panel -> User -> Create)**   
![Step 1a](nas_rsync_1a_user.png)  
  
* **Step 1b: Set User Groups (Control Panel -> User -> rsync -> User Groups)**
![Step 1b](nas_rsync_1b_user.png)  
  
* **Step 1c: Set User Permissions (Control Panel -> User -> rsync -> Permissions)**  
![Step 1c](nas_rsync_1c_user.png)  

## Step 2: NAS changes (SSH):
Step 2x: Connect to the NAS with a ssh client (I use PuTTY) and login in with the rsync user creeated in Step 1.  
![Step 2](nas_rsync_2_ssh.png)  

> * **Step 2a: "file: sshd_config"**  With Vim, edit the /etc/ssh/sshd_config file
```text
sudo vim /etc/ssh/sshd_config
```
> * **Step 2a: "file: sshd_config"**  In Vim, make these changes
```text
remove "#" before "PubkeyAuthentication yes"
remove "#" before "AuthorizedKeysFile .ssh/authorized_keys"
remove "#" before "ChallengeResponseAuthentication no"
```
> * **Step 2a: "file: sshd_config"**  In Vim, save the file
```text
:wq!
```
  
> * **Step 2b: "folder: .ssh"**  Create folder (in rsync user home folder)
```text
mkdir .ssh
```
> * **Step 2b: "folder: .ssh"**  Change folder permission to "drwx------"
```text
chmod 700 .ssh
```
  
> * **Step 2c: "file: authorized_keys"**  Create file in the .ssh folder 
```text
touch .ssh/authorized_keys
```
> * **Step 2c: "file: authorized_keys"**  Change file permission to "-rw-------" 
```text
chmod 600 .ssh/authorized_keys
```
  
> * **Step 2d: "folder: rsync homes folder"**  Print rsync home folder 
```text
pwd
```
> * **Step 2d: "folder: rsync homes folder"**   Go to homes folder 
```text
cd /var/services/homes
```
> * **Step 2d: "folder: rsync homes folder"**   Change folder permission to "drwx------" 
```text
sudo chmod 700  rsync
```




create folder ".ssh":  mkdir .ssh
chmod 700 .ssh # changes .ssh directory permission to drwx------ 
check = ls -al
create file in the .ssh folder "authorized_keys":  
touch .ssh/authorized_keys
chmod 600 .ssh/authorized_keys # changes to -rw-------
pwd = rsync's user home directory (/var/services/homes/rsync)
check = ls -al
cd /var/services/homes 
sudo chmod 700  rsync # changes rsync's home permission to drwx------
check = ls -al
activate
Control Panel -> Terminal and SNMP -> Terminal -> uncheck "Enable SSH" -> Apply
Control Panel -> Terminal and SNMP -> Terminal -> check "Enable SSH" -> Apply

## Step 3: Putty, Copy RPi to NAS: