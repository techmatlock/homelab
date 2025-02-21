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
This is what my shares look like in TrueNAS
![Screenshot from 2025-02-21 14-34-03](https://github.com/user-attachments/assets/40878da8-7f61-4437-913c-15e20fd229ae)

Add persistent mount to `/etc/fstab`

```
//10.10.10.2/media /mnt/media cifs credentials=/etc/smbcredentials,uid=3000,gid=3000,iocharset=utf8
```

Apply changes and mount

```
sudo mount -a
```
