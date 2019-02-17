---
title: Server Backups With Borg
published: true
---

I'm currently using borg to backup my laptop and server files to an off-site location hosted by [@johannwagner](https://github.com/johannwagner). I want to share my configuration which is now running successfully for about half a year.

On my systems I am not using borg directly but a python wrapper script called `borgmatic`. Borgmatic provides a simple configuration for the most common backup tasks. The script can be installed via the package manager or directly via `pip`. The configuration is located in `/etc/borgmatic/config.yaml`. By default it is well documented and easy to understand. Below I will share my bits and peaces of the configuration so you get an idea on how a production config looks like.

```yaml
location:
  source_directories:
    - /bin
    - /boot
    - /etc
    - /home
    - /lib32
    - /lib64
    - /opt
    - /root
    - /sbin
    - /usr
    - /var
  repositories:
    - user@my.server.url:/the/backup/repo/path
  exclude_patterns:
    - .cache
```

The first lines are quite straight forward, however the storage section of my configuration has some good hints. I realized that `zstd` at compression level 3 is best suited for my backups according cpu load and overall size. To let the backups run automatically borg needs to know which ssh key to use, so providing it explicitly makes borg independent of the underlying systems understanding of the ssh-key path.

```
storage:
  encryption_passphrase: "my password"
  compression: zstd,3
  ssh_command: ssh -i /path/to/ssh/key/id_rsa
```

The retention section configures which backups are stored for how long. It provides a better rotation strategy than just removing old backups.

```
retention:
  keep_daily: 7
  keep_weekly: 4
  keep_monthly: 6
  keep_yearly: 2
```

After configuring borgmatic you can run it manually with

```
$ borgmatic
```

to ensure your configuration is correct. You can check if your backup got written to the server with

```
$ borg list user@my.server.url:/the/backup/repo/path
```

When one borg action is running on a borg repository the repository gets locked and no other action can be run until the previous one finishes.

After you verified that your configuration is running you should automate things. Preferably you do this with a systemd service as a system process. I am using the following `borgmatic.service` file for that:

```
[Unit]
Description=borgmatic backup
StartLimitIntervalSec=300
StartLimitBurst=5
Wants=network-online.target
After=network-online.target

[Service]
Type=simple
Environment=SSH_AUTH_SOCK=/run/user/1000/keyring/ssh
ExecStart=/usr/bin/borgmatic
Restart=on-failure
RestartSec=60
```

This file should be located in `/etc/systemd/system/`. It handles quite a number of things which I need for my backup strategy. First, it retries to do a backup if the first attempt failed. Specifically it retries the backup 5 times every 5 minutes when the network on your system got available. Secondly, it uses your users ssh authentication socket to support the usage of ssh-keys that are encrypted with a password. By using the keyrings ssh socket, the backups are ready to be written to your server as soon as the user logs in on the login screen. For servers this is obviously not possible, you have to use tools like the ssh-agent to unlock your ssh keys and make an ssh auth socket available. Now test your service with

```
$ systemctl daemon-reload
$ systemd-analyze verify borgmatic.service
$ systemctl start borgmatic.service
```

To make this now run every day I created a systemd timer:

```
[Unit]
Description=runs borgmatic daily

[Timer]
OnCalendar=daily
Persistent=true

[Install]
WantedBy=timers.target
```

This runs the previously configured systemd service every day. Place it also in `/etc/systemd/system/` and reload the systemd daemon to discover the new timer. To enable the timer in your system do

```
$ systemctl enable borgmatic.timer
```

## Restoring Files From a Borg Backup

Before restoring files from a backup you need to know from *which* backup you want to restore your files. To display all available backups use

```
$ borg list user@my.server.url:/the/backup/repo/path
```

and look for the first column. Look at the timestamp to decide from which backup you want to restore your file.

To actually restore a file use the `extract` command:

```
$ borg extract user@my.server.url:/the/backup/repo/path::first-column-from-list my/file/to/restore.txt
```

This will restore the file to `./my/file/to/restore.txt` and not to the original location. So make sure to put the restored file to the right place after extracting it.

## Summing It Up

The described solutions provides me with automatic off-site backups that are encrypted by default and transmitted over a secure channel. I am quite happy with this setup and look forward in saving me pain :D
