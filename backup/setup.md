# backup

## backup target server

apt install rsync
#groupadd -g 801 backup
useradd -g backup -m -s /bin/bash -u 801 ubackup
su ubackup
ssh-keygen -t ed25519 -C "backup"
cat /home/ubackup/.ssh/id_ed25519.pub
#ssh-keygen -t rsa -b 2048 -C "backup-rsa"
#cat /home/ubackup/.ssh/id_rsa.pub
touch /var/log/backup.log
chown ubackup:backup /var/log/backup.log
chmod 664 /var/log/backup.log

## source server

apt install rsync sudo
#groupadd -g 801 backup
useradd -g backup -m -s /bin/bash -u 801 ubackup
#passwd ubackup
#passwd --delete ubackup
#adduser ubackup sudo
#deluser ubackup sudo
```
echo "ubackup ALL=NOPASSWD:/usr/bin/rsync" > /etc/sudoers.d/ubackup-rsync
chmod 440 /etc/sudoers.d/ubackup-rsync
```
su ubackup
mcedit ~/id_ed25519.pub
#mcedit ~/id_rsa.pub
mkdir -p ~/.ssh
cat ~/id_ed25519.pub >> ~/.ssh/authorized_keys
#cat ~/id_rsa.pub >> ~/.ssh/authorized_keys
exit

## backup target server

mcedit ~/backup-cron.sh
```
#!/bin/sh
if (/home/ubackup/backup.sh >> /var/log/backup.log 2>&1) ; then
  echo "$(date +'%Y%m%d_%H%M%S') backup success" >> /var/log/backup.log
else
  echo "$(date +'%Y%m%d_%H%M%S') backup failed" >> /var/log/backup.log
fi
```
mcedit ~/backup.sh
```
#!/bin/sh
set -eu
mkdir -p /home/ubackup/remote-backups
rsync -avR -r --delete --delete-during --rsync-path="{ echo <SUDOPASS>; cat; } | sudo -S rsync" --exclude="private.encrypted" ubackup@example.com:"/root/docker" "/home/ubackup/remote-backups"
```
chmod 755 ~/backup-cron.sh
chmod 755 ~/backup.sh