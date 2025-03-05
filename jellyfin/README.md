# jellyfin

### Install Jellyfin (for AMD)

`curl https://repo.jellyfin.org/install-debuntu.sh | sudo bash`

### Go to the host machine

1. Open config for GPU access to LXC container `/etc/pve/lxc/<container_id>.conf`

```
#Add these lines
lxc.cgroup2.devices.allow = c 226:* rwm
lxc.mount.entry = /dev/dri dev/dri none bind,optional,create=dir
```

2. Restart container

```
pct stop <container_id>
pct start <container_id>
```

### Go to the LXC container

1. Check if `/dev/dri` appears in LXC container

```
ls -l /dev/dri
```

2. Set up permissions

```
apt update && apt install -y vainfo mesa-va-drivers
usermod -aG render,video jellyfin

pct stop <container_id>
pct start <container_id>
```

3. Check if VA-API works

```
vainfo --display drm --device /dev/dri/renderD128

```

4. Configure Jellyfin to use hardware acceleration:

```
Go to Jellyfin Admin Panel → Playback

Under Hardware Acceleration, select VA-API

Set Render Device to /dev/dri/renderD128

Restart Jellyfin and check Logs for hardware acceleration.

```

Install CIFS

```
sudo apt install cifs-utils -y
```

One-time mount (without persistence)

```
sudo mount -t cifs //10.10.10.2/media /mnt/media --verbose -o credentials=/etc/smbcredentials,uid=1000,gid=1000,iocharset=utf8,noperm,vers=3.0
```

Create credentials file for persistent mount

```
sudo vim /etc/smbcredentials
# Add the following lines:
# username=your_user
# password=your_password
```

Secure the credentials file

```
sudo chmod 600 /etc/smbcredentials
```

Add persistent mount to fstab

```
sudo vim /etc/fstab
# Add this line:
# //10.10.10.2/media /mnt/media cifs credentials=/etc/smbcredentials,uid=1000,gid=1000,iocharset=utf8,noperm,vers=3.0 0 0
```

Apply changes

```
sudo mount -a
```

# Notes

I had to change unprivileged container to privileged:

```
vim /etc/pve/lxc/<container_id>.conf
change to unprivileged: 0

pct stop <container_id>
pct start <container_id>
```

View Jellyfin logs

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
