---
title: "Automatic backups over the LAN"
date: "2018-01-21"
publishDate: "2017-05-19"
tags:
  - network
  - ssh
  - crypto
  - linux
slug: "backup-over-lan"
---

:penguin: [Home Server](https://www.circuidipity.com/home-server/) :: Make incremental and automatic backups of a home folder to a local server using **SSH + rsync + cron**.

## Let's go!

**Server** is a [raised-from-the-dead netbook](https://www.circuidipity.com/laptop-home-server) on the home network. **Client** desktop is configured to perform a daily sync of its `$HOME` to server.

## 0. Server and Client: SSH keys

Create [cryptographic keys and disable password logins](https://www.circuidipity.com/secure-remote-access-using-ssh-keys) to make traffic between server and client more secure.

## 1. Server: backup directory

Create a directory to store the backup ...

```bash
mkdir -p /path/to/backup                                             
```

## 2. Client: hosts alias

Set the IP address and hostname of server in `/etc/hosts`. Sample entry ...

```bash
192.168.1.88    netbook.lan
```

## 3. Client: sync

Test syncing `$HOME/` to `netbook.lan:/path/to/backup/` with the `rsync --dry-run` option. Example ...

```bash
rsync --dry-run --archive --delete --verbose $HOME/ netbook.lan:/path/to/backup/
```

If everything checks out OK drop `--dry-run` and re-run the command to make a proper backup.

## 4. Client: script

Start piling on the options to rsync and the command quickly becomes awkward and easy to get wrong. Option `--delete` is useful but can generate unpleasant surprises. A few things I `--exclude` from a `$HOME` sync are `[Cc]ache` and `[Tt]rash` and `[Tt]humbnails`, and pay attention to the trailing forward-slash `/` on directories.

:penguin: [$HOME Slash Bin](https://www.circuidipity.com/homebin/) :: I create a shell script `teleportHome.sh` that makes use of the `keychain` utility to manage SSH keys for [password-less logins to servers](https://www.circuidipity.com/secure-remote-access-using-ssh-keys) and place in `$HOME/bin`. Sample ...

```bash
#!/bin/bash                                                                     
SYNC_OPT="--archive --verbose --delete"
EXCLUDE_OPT="--exclude=*[Cc]ache*/ --exclude=*[Tt]rash*/ --exclude=local/ \
--exclude=*[Tt]humbnail*/"
DESTINATION="netbook.lan:/path/to/backup/"

. ${HOME}/.keychain/${HOSTNAME}-sh                                              
rsync $SYNC_OPT $EXCLUDE_OPT ${HOME}/ ${DESTINATION}/
```

Source: [teleportHome.sh](https://github.com/vonbrownie/homebin/blob/master/teleportHome.sh) - A more complete script that verifies that DESTINATION exists; adds option to include (and exclude) items from a config file.

## 5. Client: automate

Automate the backups by running `crontab -e` and creating a **cron job**. Sample entry runs the backup script daily at 02:30 ...

```bash
# m h  dom mon dow   command
30 2 * * * . /home/USERNAME/.keychain/$(/bin/hostname)-sh; /home/USERNAME/bin/teleportHome.sh netbook.lan:/path/to/backup/
```

## 6. Server: rsnapshot

Use the **rsync** powers of [rsnapshot](https://www.circuidipity.com/rsnapshot/) to make daily, weekly, and monthly **incremental** backups of data.

Happy hacking!
