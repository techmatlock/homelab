1. Since we are using Ubuntu which is using systemd-resolved. We need to disable the DNS caching stub resolver to allow Pi-hole to listen on port 53.
   `sudo sed -r -i.orig 's/#?DNSStubListener=yes/DNSStubListener=no/g' /etc/systemd/resolved.conf`

2. This will not change the nameserver settings, which point to the stub resolver thus preventing DNS resolution. Change the `/etc/resolv.conf` symlink to point to `/run/systemd/resolve/resolv.conf`, which is automatically updated to follow the system's netplan: `sudo sh -c 'rm /etc/resolv.conf && ln -s /run/systemd/resolve/resolv.conf /etc/resolv.conf'` After making these changes, you should restart systemd-resolved using `systemctl restart systemd-resolved`.

3. Remove/change nameservers in `/etc/netplan/50-cloud-init.yaml` to only include localhost `127.0.0.1`. Then run `sudo netplan try` followed by `sudo netplan apply`.

4. Run the Docker Compose file:

```yaml
---
volumes:
  pihole-data: {}

services:
  pihole:
    container_name: pihole
    image: pihole/pihole:2024.07.0 #latest v5 image
    ports:
      - "53:53/tcp"
      - "53:53/udp"
      - "80:80/tcp"
      - "443:443/tcp"
    environment:
      TZ: "America/Los_Angeles"
    volumes:
      - "pihole-data:/etc/pihole"
    restart: unless-stopped
```

Troubleshooting

```
Reset password
sudo pihole -a -p
```

Since accessing homelab on a separate VLAN, make sure to go to Settings > DNS > Bind only to interface eth0

```
pihole ignoring query from non-local network 192.168.10.254
```
