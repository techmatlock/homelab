#Copy public key to server
ssh-copy-id username@remote-server-ip
or
ssh-copy-id -i ~/.ssh/custom_key.pub username@remote-server-ip

#SSH Config Template
Host shortname
HostName full.domain.com
User your-username
IdentityFile ~/.ssh/id_rsa

#Remove public keys
echo "" > ~/.ssh/authorized_keys
