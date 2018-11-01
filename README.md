# Keep SSH session using autossh and cron
## Server Side

### Add User for SSH tunneling
```
sudo adduser --system --shell /bin/false --gecos "Auto SSH" --disabled-password --home=/home/autossh autossh
```


### Generate SSH key
```
sudo -u autossh ssh-keygen -t rsa -b 4096 -f ~/.ssh/autossh
```

### Copy public key to ~/.ssh/authorized_keys
```
sudo -u autossh cp ~/.ssh/autossh.pub ~/.ssh/authorized_keys
```

### Append Follwing Configuration to /etc/ssh/sshd_config
```
Match User autossh
   #AllowTcpForwarding yes
   #X11Forwarding no
   #PermitTunnel no
   #GatewayPorts no
   AllowAgentForwarding no
   PermitOpen localhost:52443
   PermitOpen localhost:55420
   ForceCommand echo 'This account can only be used for [reason]'
```

### Restart SSH daemon
```
sudo systemctl restart ssh
```

## Client Side

### Copy SSH private key to client host

```
scp 1.2.3.4/.ssh/autossh ~/.ssh/autossh
```

### Install autossh
```
sudo apt install autossh
```

### Add SSH Client Configurations

Append following config to ~/.ssh/config

Replace 1.2.3.4 to real IP or hostname

```
Host 1.2.3.4 autossh
    Hostname 1.2.3.4
    User autossh
    SendEnv LANG LC_*
    IdentityFile ~/.ssh/autossh
    ConnectTimeout 0
    HashKnownHosts yes
    GSSAPIAuthentication yes
    GSSAPIDelegateCredentials no
    RemoteForward 2222 localhost:22
    ServerAliveInterval 30
    ServerAliveCountMax 3
```

### Test SSH connection

```
autossh -M 0 -f -N autossh
```

### Verify SSH Connection
```
netstat -lnt
```

```
ps aux |grep autossh
```

### Copy autossh-bot to ~/bin
```
mkdir -p ~/bin
cp ./bin/autossh-bot ~/bin
```


### Register cron job
```
crontab -e 
```

Schedule cron job to every 5 minutes


```
0/5 * * * * ~/bin/autossh-bot
```
