<p align="center">
    <a href=https://github.com/kabballa/una-backup/master/LICENSE" target="_blank"><img src="https://img.shields.io/badge/license-MIT-1c7ed6" alt="License" /></a>
</p>

> If you enjoy the project, please consider giving us a GitHub star ⭐️. Thank you!

## Sponsors

If you want to support our project and help us grow it, you can [become a sponsor on GitHub](https://github.com/sponsors/olariuromeo)

<p align="center">
  <a href="https://github.com/sponsors/olariuromeo">
  </a>
</p>

# 🧩 KABBALLA – UNA APP Automated Backup & Retention System

This document describes how to configure and automate daily, weekly, monthly, and annual backups for UNA-based sites using **Coozila! KABBALLA Backup System**.

## 📁 1. Directory Structure & Files

| **Path** | **Description** |
| :--- | :--- |
| `/opt/kabballa/apps/una-backup/` | **Base application directory** |
| `/opt/kabballa/apps/una-backup/data/` | **Base backup directory for all data** |
| `/opt/kabballa/apps/una-backup/data/.env` | Environment configuration file |
| `/opt/apps/una` | Root directory containing UNA sites |
| `/opt/kabballa/apps/una-backup/daily_backup.sh` | Main backup automation script |
| `/opt/kabballa/apps/una-backup/data/logs/backup_rotation.log` | Backup log file |

## ⚙️ 2. Environment Configuration (`.env`)

Create the file:

```

/opt/kabballa/apps/una-backup/data/.env

````

with this content:

```bash
# ----------------------------------------------------------------------------------#
#                                                                                   #
#   Copyright (C) 2009 - 2025 Coozila! Licensed under the MIT License.              #
#   Coozila! Team    lab@coozila.com                                                #
#                                                                                   #
# ----------------------------------------------------------------------------------#

# The base directory where all backups are stored
BASE_BACKUP_DIR="/opt/kabballa/apps/una-backup/data"

# The main directory where the sites are located
WWW_DIR="/opt/apps/una"

# Retention periods (in days)
RETENTION_DAILY_DAYS=7
RETENTION_WEEKLY_DAYS=35
RETENTION_MONTHLY_DAYS=365
# Annual backups are kept indefinitely
````

## 🧠 3. Main Script (`daily_backup.sh`)

Place this file at:

```
/opt/kabballa/apps/una-backup/daily_backup.sh
```

Make it executable:

```bash
chmod +x /opt/kabballa/apps/una-backup/daily_backup.sh
```

### 📜 Script Content (For Reference - Full script must be provided separately)

The script handles loading environment variables, setting up directories, performing the backup (files/DB), applying hard links for retention (weekly/monthly/annual), and cleaning up old files.

```bash
# ----------------------------------------------------------------------------------#
#                                                                                   #
#   Copyright (C) 2009 - 2025 Coozila! Licensed under the MIT License.              #
#   Coozila! Team    lab@coozila.com                                                #
#                                                                                   #
# ----------------------------------------------------------------------------------#

#!/bin/bash
# ... (Full script content is available in the separate daily_backup.sh file)
```

## ⏰ 4. Automate the Script with Cron

### Step A: Make Executable

```bash
chmod +x /opt/kabballa/apps/una-backup/daily_backup.sh
```

### Step B: Edit the Crontab

```bash
crontab -e
```

Add this section:

```cron
# ----------------------------------------------------------------------------------#
#                                                                                   #
#   Copyright (C) 2009 - 2025 Coozila! Licensed under the MIT License.              #
#   Coozila! Team    lab@coozila.com                                                #
#                                                                                   #
# ----------------------------------------------------------------------------------#
# The email address to receive alerts
MAILTO="your.email@example.com"

# Schedule: Runs daily at 03:00 AM
# Redirects only STDOUT (success messages) to /dev/null
# STDERR (critical errors) are emailed automatically
0 3 * * * /opt/kabballa/apps/una-backup/daily_backup.sh >>/dev/null
```

## 📧 5. Email Alert Behavior

| **Type** | **Behavior** |
| :--- | :--- |
| ✅ Success | All logs written to `$SCRIPT_LOG`. No stdout/stderr → no email. |
| ⚠️ Error | Script writes to `stderr` → cron sends an alert email to `MAILTO`. |

## 🔍 6. Verification

After execution, check:

```bash
cat /opt/kabballa/apps/una-backup/data/logs/backup_rotation.log
```

You should see:

```
===== Backup rotation completed at YYYY-MM-DD HH:MM:SS =====
```

## ✅ Summary

| **Task** | **Command** |
| :--- | :--- |
| Make script executable | `chmod +x /opt/kabballa/apps/una-backup/daily_backup.sh` |
| Test manually | `/opt/kabballa/apps/una-backup/daily_backup.sh` |
| View logs | `cat /opt/kabballa/apps/una-backup/data/logs/backup_rotation.log` |
| Edit cron | `crontab -e` |
| Edit .env | `nano /opt/kabballa/apps/una-backup/data/.env` |

> 🧠 **Tip:** Extend the backup system with `rclone` or `rsync` to replicate backups to remote storage (e.g., S3, Google Drive, Ceph Object Gateway).

## Contributing

We welcome contributions to this project\! Please refer to our [Contributing Guidelines](https://www.google.com/search?q=CONTRIBUTING.md) for detailed instructions on how to contribute.

For questions or contributions, feel free to contact the **Backup Developer** at [olariu\_romeo@yahoo.it](mailto:olariu_romeo@yahoo.it).

### Code of Conduct

We are committed to fostering an inclusive and respectful environment. Please review our [Contributor Code of Conduct](https://www.google.com/search?q=CODE_OF_CONDUCT.md) for guidelines on acceptable behavior.

## Trademarks and Copyright

This software listing is packaged by Romulus. All trademarks mentioned are the property of their respective owners, and their use does not imply any affiliation or endorsement.

### Copyright

Copyright (C) Coozila\! Licensed under the MIT License.

### Licenses

  * **Coozila\!**: [MIT License](https://github.com/kabballa/una-backup/blob/master/LICENSE)

## Disclaimer

This product is provided "as is," without any guarantees or warranties regarding its functionality, performance, or reliability. By using this product, you acknowledge that you do so at your own risk. Romulus and its contributors are not liable for any issues, damages, or losses that may arise from the use of this product. We recommend thoroughly testing the product in your own environment before deploying it in a production setting.

Happy coding\!