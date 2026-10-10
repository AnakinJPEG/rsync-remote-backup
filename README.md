# Linux Automated Remote Backup with rsync

A lightweight, automated off-site backup solution using `rsync` and `cron` to protect critical data against drive failure, accidental deletion, and ransomware.

## The Problem

Local backups are not enough. If a drive fails, a machine is stolen, or ransomware encrypts the filesystem, any backup stored on the same machine is lost with it. Manual backups are also easy to forget and rarely happen consistently.

This project solves that by automating incremental, encrypted transfers to a remote host on a fixed schedule.

## The Solution

A `rsync`-based backup script scheduled via `cron` that:

- Transfers only changed data (incremental) to minimize bandwidth and time
- Preserves file permissions, timestamps, and symlinks
- Runs unattended on a nightly schedule
- Logs each run so failures are visible

## Architecture
```
┌─────────────────┐         SSH / rsync          ┌──────────────────┐
│  Local Machine  │  ──────────────────────────► │  Remote Host     │
│  /data          │   incremental transfer       │  /backups/data   │
└─────────────────┘                              └──────────────────┘
        │
        ▼
   cron (nightly)
        │
        ▼
   backup.log
```

## How It Works

1. `cron` triggers the backup script at a defined time (e.g., 2:00 AM daily).
2. `rsync` compares source and destination, transferring only changed or new files.
3. The `--delete` flag keeps the remote mirror accurate to the source.
4. Output is appended to a log file for auditing.

## Setup

**1. Generate SSH keys for passwordless transfer:**
```bash
$ ssh-keygen                   # 2 keys will be generated, the public must be sent to the backup machine and vice versa (.pub extension)
$ ssh-copy-id <hostname:ip>  
$ ls -l .ssh/authorized_keys   # Check on both machines
```

(!) Recommended permissions for the keys would be:
Private Key - rw-------
Public Key  - rw-r--r--

**2. Create the backup script (`backup.sh`):**
```bash
crontab -u <user> -e
```
*In the user-specified crontab*
```
0 2 * * * rsync -avz --delete /path/to/source/ user@remote:/path/to/backup/ >> /home/youruser/backup.log 2>&1
```

"https://crontab-generator.org" for a more complex schedule assistance

## Verification

Check the log after the first run:
```bash
cat /home/youruser/backup.log
```

## What I Learned

- How `rsync` performs delta transfers and why that matters for large datasets
- The importance of `--delete` for maintaining a true mirror (and its risks)
- Scheduling and logging unattended jobs with `cron`
- Hardening SSH access with key-based authentication

## Requirements

- Linux/Unix system with `rsync` installed
- SSH access to a remote host
- Basic `cron` availability

## License

MIT

