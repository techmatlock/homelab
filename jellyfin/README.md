# jellyfin

### Proxmox Host

1. Open the shell on your Proxmox node and find out the GID for video and render groups using the command cat /etc/group

- Find video and render in the output. It should look something like this video:x:44: and render:x:104: Note the numbers 44 and 104.
- Type this command and find what video and render devices you have ls /dev/dri/ . If you only have an iGPU, you may see cardx and renderDy in the output. If you have an iGPU and a dGPU, you may see cardx1, cardx2 and renderDy1 and renderDy2 . Here x may be 0 or 1 or 2 and y may be 128 or 129. (This guide only focuses on iGPU pass through but you may be able to passthrough a dGPU in a similar manner. I just haven't done it and I am not a 100% sure it would work.)
- We need to pass the cardx and renderD devices to the lxc. Note down these devices
- A note that the value of cardx and renderD may not always be the same after a server reboot. If you reboot the server, repeat steps 3 and 4 below.

2. Go to your container and in the resources tab, select Add -> Device Passthrough .
3. In the device path add the path of cardx - /dev/dri/cardx
4. In the GID in CT field, enter the number that you found in step 1 for video group. In my case, it is 44. Hit OK
   Follow the same procedure as step 3 but in the device path, add the path of renderDy group (/dev/dri/renderDy) and in the GID field, add the ID associated with the render group (104 in my case)

5. Start your container and go to the container console. Check that both the devices are now available using the command ls /dev/dri

### Jellyfin Hardware Acceleration (AMD iGPU)

https://jellyfin.org/docs/general/administration/hardware-acceleration/amd

### NFS Share Mount Instructions

1. Install CIFS

```
sudo apt install cifs-utils -y
```

2. One-time mount (without persistence)

```
sudo mount -t cifs //10.10.10.2/media /mnt/media --verbose -o credentials=/etc/smbcredentials,uid=1000,gid=1000,iocharset=utf8,noperm,vers=3.0
```

3. Create credentials file for persistent mount

```
sudo vim /etc/smbcredentials
# Add the following lines:
# username=your_user
# password=your_password
```

4. Secure the credentials file

```
sudo chmod 600 /etc/smbcredentials
```

5. Add persistent mount to fstab

```
sudo vim /etc/fstab
# Add this line:
# //10.10.10.2/media /mnt/media cifs credentials=/etc/smbcredentials,uid=1000,gid=1000,iocharset=utf8,noperm,vers=3.0 0 0
```

6. Apply changes

```
sudo mount -a
```

### Troubleshooting

Add jellyfin user to render/video group

```
usermod -aG render jellyfin
usermod -aG video jellyfin
systemctl restart jellyfin

```

```
journalctl -u jellyfin --no-pager --lines=50
```

Fix directory permissions

```
chown -R jellyfin:jellyfin /var/lib/jellyfin
chmod -R 755 /var/lib/jellyfin
```

Restart Jellyfin

```
systemctl daemon-reload
systemctl restart jellyfin
systemctl status jellyfin
```
