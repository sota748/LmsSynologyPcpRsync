# rsync - NAS
[Return to README](../README.md)

## Gathering knowledge:
To understand how rsync works on a Synology NAS, I have found good information at the links below. (rigth click and "Open in a new Tap/Window") 
- **Synology forum:** How to enable SSH key authentication on Synology NAS, https://community.synology.com/enu/forum/1/post/136213  
- **YouTube:** Enable Key Based SSH Authentication For Synology Servers, https://www.youtube.com/watch?v=XN9SuzV08Ew  
- **YouTube:** Use a RaspberryPi as an Offsite / Remote Backup for your Critical Data!, https://www.youtube.com/watch?v=ffx4HhNLqfI  

## Step 1: DSM 7.x, User:
I have created a new user "rsync" on the NAS which is only used for rsync.  
The "rsync" user belongs to the Admin group (only Admins can SSH)  
Give the "rsync" user the desired Permissions  
  
Step 1a: Create NAS user (Control Panel -> User -> Create)   
![Step 1a](nas_rsync_1a_user.png)  
  
Step 1b: Set User Groups (Control Panel -> User -> rsync -> User Groups)
![Step 1b](nas_rsync_1b_user.png)  
  
Step 1c: Set User Permissions (Control Panel -> User -> rsync -> Permissions)  
![Step 1c](nas_rsync_1c_user.png)  

## Step 2: Putty, NAS:

## Step 3: Putty, Copy RPi to NAS: