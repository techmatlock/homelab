# jellyfin

Install CIFS

```
sudo apt install cifs-utils
```

Create local CIFS share

```
mkdir -p /mnt/media
```

Connect CIFS share

```
mount -t cifs //10.10.10.2/media /mnt/media --verbose -o user=smbuser
```

Add persistent mount to `/etc/fstab`

```
//10.10.10.2/media /mnt/media cifs credentials=/etc/smbcredentials,uid=3000,gid=3000,iocharset=utf8
```

Apply changes and mount

```
sudo mount -a
```
