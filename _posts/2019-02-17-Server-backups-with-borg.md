---
title: Server Backups With Borg
published: false
---

`/etc/systemd/system/borgmatic.service`

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

`/etc/systemd/system/borgmatic.timer`

```
[Unit]
Description=runs borgmatic daily

[Timer]
OnCalendar=daily
Persistent=true

[Install]
WantedBy=timers.target
```

`/etc/borgmatic/config.yaml`

```
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

```
storage:
  encryption_passphrase: "my password"
  compression: zstd,3
  ssh_command: ssh -i /path/to/ssh/key/id_rsa
```

```
retention:
  keep_daily: 7
  keep_weekly: 4
  keep_monthly: 6
  keep_yearly: 2
```

When one borg action is running on a borg repository the repository gets locked and no other action can be run until the previous one finishes.

Restoring files from a backup

