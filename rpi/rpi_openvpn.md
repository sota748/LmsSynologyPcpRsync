[Return to README](../README.md)  

# OpenVPN - Raspberry Pi  
The following files will be stored in directory /home/tc/  

## Step 1: RPI changes (SSH):  
**Step 1a: Connect to the RPi with a ssh client**  (I use PuTTY) and login in with the rsync user creeated in Step 1.  

**Step 1b: VPN.conf**  Create file + Contens + Make file executable + Backup  
>Create file  
```text
sudo nano VPN.conf
```
>Contents  
Replace <DDNS> with your own DDNS
Replace <PORT> with your own DDNS
Replace <NAS_IP> with your own nas ip

```text
dev tun
tls-client

remote <DDNS>.synology.me <PORT>

# The "float" tells OpenVPN to accept authenticated packets from any address,
# not only the address which was specified in the --remote option.
# This is useful when you are connecting to a peer which holds a dynamic address
# such as a dial-in user or DHCP client.
# (Please refer to the manual of OpenVPN for more information.)

#float

# If redirect-gateway is enabled, the client will redirect it's
# default network gateway through the VPN.
# It means the VPN connection will firstly connect to the VPN Server
# and then to the internet.
# (Please refer to the manual of OpenVPN for more information.)

#redirect-gateway def1

# dhcp-option DNS: To set primary domain name server address.
# Repeat this option to set secondary DNS server addresses.

#dhcp-option DNS DNS_IP_ADDRESS

pull
route-nopull
route <NAS_IP> 255.255.255.0
#route <NAS_IP> 255.255.255.0

# If you want to connect by Server's IPv6 address, you should use
# "proto udp6" in UDP mode or "proto tcp6-client" in TCP mode
proto udp

script-security 2


comp-lzo

reneg-sec 0

# Clients running OpenVPN 2.4 and higher will automatically upgrade from AES-256-CBC to AES-256-GCM without any configuration changes.
data-ciphers AES-256-CBC
cipher AES-256-CBC
auth SHA512

auth-user-pass /home/tc/VPN.key
auth-nocache
<ca>
-----BEGIN CERTIFICATE-----
xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
-----END CERTIFICATE-----

</ca>
key-direction 1
<tls-auth>
#
# 2048 bit OpenVPN static key
#
-----BEGIN OpenVPN Static key V1-----
yyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyy
yyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyy
-----END OpenVPN Static key V1-----

</tls-auth>
tc@pcp3:~$
```

* **VPN.key**  ?????  
Replace <USER> with your own DDNS
Replace <PASSWORD> with your own DDNS

```text
<USER>
<PASSWORD>
```

* **VPNstartscript.sh**  ?????  
Replace <NAS_IP> with your own nas ip

```text
#!/bin/sh

## For VPNTest
if [ -f /home/tc/TestingVPN ] ; then rm -f /home/tc/TestingVPN ; fi                                             # At startup if file exists remove it
if [ -f /home/tc/VPN_Retries ] ; then rm -f /home/tc/VPN_Retries ; fi                                           # At startup if file exists remove it
if [ -f /home/tc/VPN_Retry_Details.txt ] ; then rm -f /home/tc/VPN_Retry_Details.txt ; fi                       # At startup if file exists remove it
let retries=0
touch /home/tc/VPN_Retries                                                                                      # At startup create file
touch /home/tc/VPN_Retry_Details.txt                                                                            # At startup create file

Testhost=<NAS_IP>                                                                                              # Set a host on the local (VPN server) LAN to ping to check whether VPN is up

VPNTest() {

while true
  do

    sleep 60                                                                                                    # Wait 60 seconds for network to be "Up"

    if [ -f /home/tc/TestingVPN ]; then                                                                         # If VPN is being tested don't start again
      echo "VPN is already being tested"
    else

      touch /home/tc/TestingVPN

      homenet=$(/sbin/ifconfig | grep -Eo 'inet (addr:)?([0-9]*\.){3}[0-9]*' | grep -Eo '([0-9]*\.){2}[0-9]*' | grep -v '127.0.0')


      if [ "$homenet" != "10.44.1" ]; then
        count=$(ping -c 4 $Testhost | grep 'received' | awk -F',' '{ print $2 }' | awk '{ print $1 }')
        if [ $count -eq 4 ]; then
          sleep 1
        else
          retries=$(( retries + 1 ))
          if pgrep openvpn &> /dev/null ; then sudo killall openvpn ; fi                                        # If VPN is down and "Testhost" can't be pinged restart openvpn client
            sleep 10
            sudo openvpn --config /home/tc/VPN.conf > /dev/null 2>&1 &                                          # Replace 'openvpn --config /home/tc/VPN.conf' with your command to start openvpn
            echo $retries > /home/tc/VPN_Retries
            echo $(date +%d/%m/%Y"  "%H:%M:%S) "Retry " $retries >> /home/tc/VPN_Retry_Details.txt              # If VPN is down and "Testhost" can't be pinged add details to /home/tc/VPN_Retry_Details.txt
          fi
      else
        sleep 1
      fi

  rm -f /home/tc/TestingVPN                                                                                     # Once testing is complete remove file

    fi

  sleep 840                                                                                                     # Sleep for 14 minutes

  done
}

VPNTest &                                                                                                       # Run VPNtest loop
exit

```

* **VPN_Retries**  Example of Content  
```text
tc@pcp3:~$ cat VPN_Retries
2
```

* **VPN_Retry_Details.txt**  Example of Content  
```text
tc@pcp3:~$ cat VPN_Retry_Details.txt
14/07/2026 10:23:51 Retry  1
11/08/2026 16:25:54 Retry  2
```

* **home dir**  Content  
```text
tc@pcp3:~$ ls -la
total 68
drwxr-s--x    5 tc       staff          400 Aug 17 10:17 ./
drwxrwxr-x    3 root     staff           60 Jan  6  2017 ../
drwxr-xr-x    2 tc       staff           40 Feb  1  2025 .X.d/
-rw-r--r--    1 tc       staff          114 Feb  1  2025 .alsaequal.presets
-rw-rw-r--    1 tc       staff         6747 Aug 17 10:28 .ash_history
-rw-r--r--    1 tc       staff         1016 Feb  1  2025 .ashrc
drwxr-s---    3 tc       staff           60 Jan  6  2017 .local/
-rw-rw-r--    1 tc       staff          920 Aug 20  2023 .profile
-rw-r--r--    1 root     staff           62 Jun  3  2025 .slimserver.cfg
drwx--S---    2 tc       staff          120 Jan  1  1970 .ssh/
-rw-r--r--    1 tc       staff         1596 Aug 17 03:03 RSYNClog.txt
-rwxr-xr-x    1 tc       staff         2773 Jun 25  2025 RSYNCstartscript.sh
-rw-r--r--    1 root     staff         5773 Aug  6  2025 VPN.conf
-r-S--S---    1 root     staff           19 Jun  2  2025 VPN.key
-rw-r--r--    1 root     staff            2 Aug 11 16:25 VPN_Retries
-rw-r--r--    1 root     staff           58 Aug 11 16:25 VPN_Retry_Details.txt
-rwxr-xr-x    1 root     staff         2169 Jun  2  2025 VPNstartscript.sh
-rwxr-xr-x    1 tc       staff         2371 Feb  1  2025 pcp-powerbutton.sh
-rwxr-xr-x    1 tc       staff         2371 Feb  1  2025 pcp-powerbutton.sh.sample
-rwxr-xr-x    1 tc       staff          713 Feb  1  2025 powerscript.sh
```
