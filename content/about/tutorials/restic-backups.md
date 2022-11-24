---
title: "Creating Backups with Restic and B2 Cloud"
date: 2020-12-07T10:02:48-05:00
draft: false
---

### Reliable Backups on a Linux System with Restic Backblaze B2 and systemd

Backups are a critical part of ensuring your data stays safe. Losing data is not something anyone wants to experience and reliable backups are how to ensure this never happens.

Backup infrastructure is critical but shouldn't require large amounts of time to maintain. A program to create backups should work quietly in the background while ensuring data integrity and safety.

Restic is a great backup tool with powerful features like deduplication and encryption. It supports B2 cloud as a storage backend allowing offsite backups for an inexpensive price. Restic is a CLI tool and doesn't include any scheduled tasks to run to create backups regularly. This tutorial configures systemd to run daily backups, prune the backup repository and run data integrity checks periodically.

Restic has many backends capable of storing backup data. Backblaze B2 is a good option due to low cost of storage and availability of the stored data.

### Creating the config file

This file contains the authentication details as well as any settings to provide to the Restic command. Save the file with restrictive permissions only readable by the root user to prevent leaking the credentials.

``` bash
❯ sudo cat /etc/restic-backup.conf
BACKUP_PATHS="/etc/restic.includes"
EXCLUDE_PATHS="/etc/restic.excludes"
RETENTION_DAYS=7
RETENTION_WEEKS=8
RETENTION_MONTHS=12
RETENTION_YEARS=10
B2_ACCOUNT_ID=""
B2_ACCOUNT_KEY=""
RESTIC_REPOSITORY="b2:[REPO_NAME]:/"
RESTIC_PASSWORD=""
RESTIC_CACHE_DIR=/tmp/restic_cache
```

Configure the include and exclude files to specify which files to backup.

``` bash
❯ sudo cat /etc/restic.includes
/home/mitch
```

``` bash
❯ sudo cat /etc/restic.excludes
**/node_modules/**
**.local/share/Steam/**
**/.stversions/**
**/.PlayOnLinux/**
**/.local/share/Trash/**
**/downloads/**
```

### Configuring systemd

This backup configuration consists of three systemd service units, 'restic-backup', 'restic-prune', and 'restic-check'. These three units each run a specific job to maintain the Restic repository. There are separate timer unit files to run the backup tasks periodically. Once the unit files are in place, use the `systemctl` command to activate and start the timer services.

#### restic-backup

The `restic-backup.service` unit handles creating the backup snapshots.

``` bash
❯ cat /etc/systemd/system/restic-backup.service
[Unit]
Description=Restic system backup
Before=restic-prune.service
Wants=restic-prune.service
JoinsNamespaceOf=restic-prune.service restic-check.service

[Service]
Type=oneshot
ExecStart=restic backup --verbose --tag auto-backup --iexclude-file $EXCLUDE_PATHS --files-from $BACKUP_PATHS ; /usr/bin/sleep 20
EnvironmentFile=/etc/restic-backup.conf
SuccessExitStatus=3
```

The backup frequency is configured with the `restic-backup.timer` unit file.

``` bash
❯ cat /etc/systemd/system/restic-backup.timer
[Unit]
Description=Backup with restic daily

[Timer]
OnCalendar=daily
RandomizedDelaySec=6hours
Persistent=true

[Install]
WantedBy=timers.target
```

#### restic-prune

The `restic-prune.service` unit runs the prune command on the repo, using the retention periods configured in the `restic-backup.conf` file. This unit is configured to run after the backup unit.

``` bash
❯ cat /etc/systemd/system/restic-prune.service
[Unit]
Description=Restic prune to clean up old backups
Requires=restic-backup.service
After=restic-backup.service
JoinsNamespaceOf=restic-backup.service restic-check.service

[Service]
Type=oneshot
ExecStart=restic forget --prune -o b2.connections=10 --compact --tag auto-backup --cleanup-cache --keep-daily $RETENTION_DAYS --keep-weekly $RETENTION_WEEKS --keep-monthly $RETENTION_MONTHS --keep-yearly $RETENTION_YEARS
EnvironmentFile=/etc/restic-backup.conf
```

#### restic-check
##### Checking backup archive integrity

The `restic-check.service` unit runs the consistency check on the repo to ensure data integrity. It's configured to use the local cache in order to reduce the download API calls to B2 cloud to lower costs.

``` bash
❯ cat /etc/systemd/system/restic-check.service
Description=Restic system backup repository consitency check
Conflicts=restic-backup.service restic-prune.service
JoinsNamespaceOf=restic-backup.service restic-prune.service
After=restic-prune.service

[Service]
Type=oneshot
ExecStart=restic --verbose=3 check --with-cache
EnvironmentFile=/etc/restic-backup.conf
```

This unit is configured to run weekly.

``` bash
❯ cat /etc/systemd/system/restic-check.timer
[Unit]
Description=Check restic repository consistency

[Timer]
OnCalendar=weekly
RandomizedDelaySec=1day
Persistent=true

[Install]
WantedBy=timers.target
```

### Activating systemd units

After configuring the unit files the systemctl configuration will need to be reloaded, then the timer units activated and started.

``` bash
systemctl daemon-reload
systemctl enable restic-backup.timer restic-check.timer
systemctl start restic-backup.timer restic-check.timer
```

The system should now create daily backups, prune any extra data daily, and check the repository consistency weekly.

Check the backup timer status with `systemctl list-timers` and individual units with `systemctl status restic-backup`.
