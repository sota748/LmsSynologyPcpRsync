[Return to README](../README.md)  

# rsync - Raspberry Pi  

## Step 1: RPI changes (SSH):  
* **Step 1a: Connect to the RPi with a ssh client**  (I use PuTTY) and login in with the rsync user creeated in Step 1.  

* **Step 1b: User "tc" must own all folders and files in /mnt/hd/Music**  for Rsync to be able to update the files' timestamps  
>Grants full ownership of the Music folder and all its contents to the user 'tc' and group 'staff'.
```text
sudo chown -R tc:staff /mnt/hd/Music
```
>Opens up the Music folder and all its contents completely, allowing anyone or any system service to read, write, or delete files.
```text
sudo chmod -R 777 /mnt/hd/Music
```
>Check, (result = "drwxrwxrwx   10 tc       staff         4096 Aug 29 10:20 /mnt/hd/Music/")
```text
sudo chown -R tc:staff /mnt/hd/Music
```
>Backup
```text
pcp bu
```
* **Step 1c: RSYNCstartscript.sh**  Create file + Contens + Make file executable + Backup  
>Create file  
```text
sudo nano RSYNCstartscript.sh
```
>Contents  
Replace <remote_source> with your own nas connection string  
Replace <nas_ip> with your own nas ip  
```text
#!/bin/sh
export HOME="/home/tc"

# Configurable paths and commands
LOG="/home/tc/RSYNClog.txt"
TMPLOG="/home/tc/RSYNClog.tmp"
RSYNC_TMP_OUTPUT="/home/tc/RSYNCoutput.tmp"
SRC=<remote_source>
DEST="/mnt/hd"
RSYNC_BIN="/usr/local/bin/rsync"
PING_BIN="/bin/ping"
ECHO_BIN="/bin/echo"
NAS_IP=<nas_ip>

# Script start time
$ECHO_BIN "[START] $(date)" >> $LOG

# Detect how the script was started
if [ -n "$SSH_CONNECTION" ]; then
  $ECHO_BIN "  INFO - Started by: SSH (PuTTY)" >> $LOG
else
  $ECHO_BIN "  INFO - Started by: CRON (configured on pCP web interface)" >> $LOG
fi

# Check ownership of destination folder
OWNER=$(ls -ld "$DEST/Music" | awk '{print $3}')
GROUP=$(ls -ld "$DEST/Music" | awk '{print $4}')
if [ "$OWNER" != "tc" ] || [ "$GROUP" != "staff" ]; then
  $ECHO_BIN "  WARNING - Ownership or group of $DEST/Music is not tc:staff (owner: $OWNER, group: $GROUP)" >> $LOG
fi

# Check if NAS is reachable
$PING_BIN -c 1 -W 5 $NAS_IP > /dev/null 2>&1
if [ $? -ne 0 ]; then
  $ECHO_BIN "[ERROR] VPN/NAS not reachable at $(date)" >> $LOG
  exit 1
fi

# Run rsync with itemize-changes and stats, but capture output to temporary file
START_TIME=$(date +%s)
$RSYNC_BIN -a --delete --itemize-changes --stats -e "ssh -i /home/tc/.ssh/id_rsa -o StrictHostKeyChecking=accept-new" "$SRC" "$DEST" > "$RSYNC_TMP_OUTPUT" 2>&1
RSYNC_EXIT=$?
END_TIME=$(date +%s)
DURATION=$((END_TIME - START_TIME))

# Analyze rsync output
CHANGES=$(grep -E '^>f|^\*deleting' "$RSYNC_TMP_OUTPUT")
CREATED=$(grep 'Number of created files:' "$RSYNC_TMP_OUTPUT" | awk '{print $5}')
DELETED=$(grep 'Number of deleted files:' "$RSYNC_TMP_OUTPUT" | awk '{print $5}')
TRANSFERRED=$(grep 'Number of regular files transferred:' "$RSYNC_TMP_OUTPUT" | awk '{print $6}')

# Logging decision
if [ $RSYNC_EXIT -eq 0 ]; then
  if [ "$CREATED" != "0" ] || [ "$DELETED" != "0" ] || [ "$TRANSFERRED" != "0" ] || [ -n "$CHANGES" ]; then
    $ECHO_BIN "  INFO - Changes detected (created: $CREATED, deleted: $DELETED, transferred: $TRANSFERRED), triggering 'pcp rescan' (RSYNC duration: ${DURATION} sec)" >> $LOG
    pcp rescan
  else
    $ECHO_BIN "  INFO - No changes detected (RSYNC duration: ${DURATION} sec)" >> $LOG
  fi
  $ECHO_BIN "[FINISH] $(date)" >> $LOG
else
  $ECHO_BIN "[ERROR] rsync failed at $(date)" >> $LOG
fi

# Keep only last 7 sync sessions in the log
awk '
  BEGIN { block = ""; count = 0; max = 7 }
  /^\[START\]/ {
    if (block != "") { sets[++count] = block }
    block = $0 "\n"
    next
  }
  { block = block $0 "\n" }
  END {
    if (block != "") { sets[++count] = block }
    start = (count > max) ? count - max + 1 : 1
    for (i = start; i <= count; i++) {
      printf "%s", sets[i]
    }
  }
' "$LOG" > "$TMPLOG"

mv "$TMPLOG" "$LOG"
rm -f "$RSYNC_TMP_OUTPUT"
```
>Make file executable  
```text
sudo chmod +x /home/tc/ RSYNCstartscript.sh
```
>Backup  
```text
pcp bu
```
* **Step 1d: RSYNClog.txt**  
>Create file  
```text
touch RSYNClog.txt
```
>Backup  
```text
pcp bu
```
>Example of Content (after )    
```text
tc@pcp3:~$ cat RSYNClog.txt
[START] Tue Aug 11 03:03:00 CEST 2026
  INFO - Started by: CRON (configured on pCP web interface)
  INFO - No changes detected (RSYNC duration: 1 sec)
[FINISH] Tue Aug 11 03:03:01 CEST 2026
[START] Wed Aug 12 03:03:00 CEST 2026
  INFO - Started by: CRON (configured on pCP web interface)
  INFO - No changes detected (RSYNC duration: 2 sec)
[FINISH] Wed Aug 12 03:03:02 CEST 2026
[START] Thu Aug 13 03:03:00 CEST 2026
  INFO - Started by: CRON (configured on pCP web interface)
  INFO - Changes detected (created: 27, deleted: 0, transferred: 24), triggering 'pcp rescan' (RSYNC duration: 48 sec)
[FINISH] Thu Aug 13 03:03:48 CEST 2026
[START] Fri Aug 14 03:03:00 CEST 2026
  INFO - Started by: CRON (configured on pCP web interface)
  INFO - Changes detected (created: 28, deleted: 0, transferred: 27), triggering 'pcp rescan' (RSYNC duration: 55 sec)
[FINISH] Fri Aug 14 03:03:55 CEST 2026
[START] Sat Aug 15 03:03:00 CEST 2026
  INFO - Started by: CRON (configured on pCP web interface)
  INFO - Changes detected (created: 60, deleted: 23, transferred: 54), triggering 'pcp rescan' (RSYNC duration: 142 sec)
[FINISH] Sat Aug 15 03:05:22 CEST 2026
[START] Sun Aug 16 03:03:00 CEST 2026
  INFO - Started by: CRON (configured on pCP web interface)
  INFO - No changes detected (RSYNC duration: 7 sec)
[FINISH] Sun Aug 16 03:03:07 CEST 2026
[START] Mon Aug 17 03:03:00 CEST 2026
  INFO - Started by: CRON (configured on pCP web interface)
  INFO - Changes detected (created: 27, deleted: 0, transferred: 23), triggering 'pcp rescan' (RSYNC duration: 55 sec)
[FINISH] Mon Aug 17 03:03:55 CEST 2026
```

* **Home dir**  
>Content  
```text
tc@pcp4:~$ ls -la
total 32
drwxr-s--x    4 tc       staff          260 Sep  1 12:02 ./
drwxrwxr-x    3 root     staff           60 Jan  6  2017 ../
drwxr-xr-x    2 tc       staff           40 Feb 28  2026 .X.d/
-rw-r--r--    1 tc       staff          114 Feb 28  2026 .alsaequal.presets
-rw-rw-r--    1 tc       staff          332 Sep  1 12:06 .ash_history
-rw-r--r--    1 tc       staff         1191 Feb 28  2026 .ashrc
drwxr-s---    3 tc       staff           60 Jan  6  2017 .local/
-rw-rw-r--    1 tc       staff          920 Aug 20  2023 .profile
-rw-r--r--    1 root     staff           62 Sep  1 01:26 .slimserver.cfg
-rw-r--r--    1 tc       staff            0 Sep  1 12:02 RSYNClog.txt
-rwxr-xr-x    1 root     staff         2779 Sep  1 11:51 RSYNCstartscript.sh
-rwxr-xr-x    1 tc       staff         2371 Feb 28  2026 pcp-powerbutton.sh
-rwxr-xr-x    1 tc       staff          713 Feb 28  2026 powerscript.sh
tc@pcp4:~$
```

## Step 2: rsync Key:  
See................