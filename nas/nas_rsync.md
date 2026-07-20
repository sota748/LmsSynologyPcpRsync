# rsync - NAS
[Return to README](../README.md)

## Gathering knowledge:
To understand how rsync works on a Synology NAS, I have found good information at the links below. (rigth click and "Open in a new Tap/Window") 
- **Synology forum:** How to enable SSH key authentication on Synology NAS, https://community.synology.com/enu/forum/1/post/136213  
- **YouTube:** Enable Key Based SSH Authentication For Synology Servers, https://www.youtube.com/watch?v=XN9SuzV08Ew  
- **YouTube:** Use a RaspberryPi as an Offsite / Remote Backup for your Critical Data!, https://www.youtube.com/watch?v=ffx4HhNLqfI  

## Step 1: User:
I have created a new user "rsync" on the NAS which is only used for rsync.  
The "rsync" user belongs to the Admin group (only Admins can ssh)  
Control Panel -> User -> rsync -> User Groups:
verify "users" and "Administrators" are checked (only Admins can ssh)  
![Step 1](nas_rsync_1_user.png)  
