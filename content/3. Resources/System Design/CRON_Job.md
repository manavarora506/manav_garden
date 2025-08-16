---
title: CRONJob
date: 2025-04-14
tags:
  - seed
enableToc: true
---

# Cron Jobs Explained

A cron job is a scheduled task in Unix-like operating systems that runs automatically at specified intervals. These jobs are managed by the cron daemon, which executes commands according to instructions in configuration files called "crontabs."

## Key Components

- **Cron daemon**: Background service that checks for scheduled tasks
- **Crontab file**: Configuration file containing scheduled commands
- **Cron expression**: Timing syntax that defines when jobs run (minute, hour, day, month, weekday)

## Cron Expression Format

```
* * * * * command_to_execute
↑ ↑ ↑ ↑ ↑
│ │ │ │ └─── Day of week (0-7, where 0 and 7 are Sunday)
│ │ │ └───── Month (1-12)
│ │ └─────── Day of month (1-31)
│ └───────── Hour (0-23)
└─────────── Minute (0-59)
```

## Examples of Cron Jobs

1. **Daily Database Backup at 2 AM**:
   ```bash
   0 2 * * * /usr/local/bin/backup_database.sh
   ```

2. **System Update Every Monday at 3 AM**:
   ```bash
   0 3 * * 1 apt-get update && apt-get -y upgrade
   ```

3. **Log Rotation Every Hour**:
   ```bash
   0 * * * * /usr/sbin/logrotate /etc/logrotate.conf
   ```

4. **Website Monitoring Every 5 Minutes**:
   ```bash
   */5 * * * * curl -s https://example.com > /dev/null || mail -s "Website Down" admin@example.com
   ```

5. **Monthly Sales Report on 1st Day**:
   ```bash
   0 7 1 * * /usr/local/bin/generate_monthly_report.py
   ```

6. **Clear Temporary Files Every Sunday**:
   ```bash
   0 0 * * 0 find /tmp -type f -atime +7 -delete
   ```

7. **Run Scripts at System Reboot**:
   ```bash
   @reboot /path/to/startup_script.sh
   ```

Cron jobs are essential for automating routine maintenance tasks, scheduled backups, report generation, and system monitoring without requiring manual intervention.