# 🧩 KABBALLA – Automated Backup & Retention System

This document describes how to configure and automate daily, weekly, monthly, and annual backups for UNA-based sites using **Coozila! KABBALLA Backup System**.


---

## 📁 1. Directory Structure & Files

| Path                                                  | Description                    |
| ----------------------------------------------------- | ------------------------------ |
| `/opt/kabballa/data/backups/`                         | Base backup directory          |
| `/opt/kabballa/data/backups/.env`                     | Environment configuration file |
| `/opt/apps/una`                                       | Root directory with UNA sites  |
| `/opt/kabballa/scripts/daily_backup.sh`               | Main backup automation script  |
| `/opt/kabballa/data/backups/logs/backup_rotation.log` | Backup log file                |

---

## ⚙️ 2. Environment Configuration (`.env`)

Create the file:

```
/opt/kabballa/data/backups/.env
```

with this content:

```bash
# ----------------------------------------------------------------------------------#
#                                                                                   #
#   Copyright (C) 2009 - 2025 Coozila! Licensed under the MIT License.              #
#   Coozila! Team    lab@coozila.com                                                #
#                                                                                   #
# ----------------------------------------------------------------------------------#

# The base directory where all backups are stored
BASE_BACKUP_DIR="/opt/kabballa/data/backups"

# The main directory where the sites are located
WWW_DIR="/opt/apps/una"

# Retention periods (in days)
RETENTION_DAILY_DAYS=7
RETENTION_WEEKLY_DAYS=35
RETENTION_MONTHLY_DAYS=365
# Annual backups are kept indefinitely
```

---

## 🧠 3. Main Script (`daily_backup.sh`)

Place this file at:

```
/opt/kabballa/scripts/daily_backup.sh
```

Make it executable:

```bash
chmod +x /opt/kabballa/scripts/daily_backup.sh
```

### 📜 Script Content

```bash
# ----------------------------------------------------------------------------------#
#                                                                                   #
#   Copyright (C) 2009 - 2025 Coozila! Licensed under the MIT License.              #
#   Coozila! Team    lab@coozila.com                                                #
#                                                                                   #
# ----------------------------------------------------------------------------------#

#!/bin/bash

# ==============================================================================
# 1. Load Environment Variables (.env)
# ==============================================================================
ENV_FILE="/opt/kabballa/data/backups/.env"

if [ -f "$ENV_FILE" ]; then
    source "$ENV_FILE"
else
    echo "CRITICAL ERROR: .env file not found at $ENV_FILE. Using defaults." >&2
    BASE_BACKUP_DIR="/opt/kabballa/data/backups"
    WWW_DIR="/opt/apps/una"
    RETENTION_DAILY_DAYS=7
    RETENTION_WEEKLY_DAYS=35
    RETENTION_MONTHLY_DAYS=365
fi

# ==============================================================================
# 2. Setup Directories and Date Variables
# ==============================================================================
DAILY_DIR="$BASE_BACKUP_DIR/daily"
WEEKLY_DIR="$BASE_BACKUP_DIR/weekly"
MONTHLY_DIR="$BASE_BACKUP_DIR/monthly"
ANNUAL_DIR="$BASE_BACKUP_DIR/annual"
SCRIPT_LOG="$BASE_BACKUP_DIR/logs/backup_rotation.log"

DATE=$(date +%F)
DAY_OF_WEEK=$(date +%u)
DAY_OF_MONTH=$(date +%d)
DAY_OF_YEAR=$(date +%j)

mkdir -p "$DAILY_DIR/html" "$DAILY_DIR/db" \
         "$WEEKLY_DIR/html" "$WEEKLY_DIR/db" \
         "$MONTHLY_DIR/html" "$MONTHLY_DIR/db" \
         "$ANNUAL_DIR/html" "$ANNUAL_DIR/db" \
         "$(dirname "$SCRIPT_LOG")"

echo "===== Backup rotation started at $(date) =====" >> "$SCRIPT_LOG"

# ==============================================================================
# 3. Functions
# ==============================================================================

link_backups() {
    local SITE_URL="$1"
    local SOURCE_FILE="$2"
    local FILE_TYPE="$3"

    if [ "$DAY_OF_WEEK" -eq 7 ]; then
        DEST_FILE="$WEEKLY_DIR/$FILE_TYPE/${SITE_URL}-$DATE.tar.gz"
        ln "$SOURCE_FILE" "$DEST_FILE"
        echo "  🔗 Hard-linked $FILE_TYPE for WEEKLY retention." >> "$SCRIPT_LOG"
    fi

    if [ "$DAY_OF_MONTH" -eq 01 ]; then
        DEST_FILE="$MONTHLY_DIR/$FILE_TYPE/${SITE_URL}-$DATE.tar.gz"
        ln "$SOURCE_FILE" "$DEST_FILE"
        echo "  🔗 Hard-linked $FILE_TYPE for MONTHLY retention." >> "$SCRIPT_LOG"
    fi

    if [ "$DAY_OF_YEAR" -eq 001 ]; then
        DEST_FILE="$ANNUAL_DIR/$FILE_TYPE/${SITE_URL}-$DATE.tar.gz"
        ln "$SOURCE_FILE" "$DEST_FILE"
        echo "  🔗 Hard-linked $FILE_TYPE for ANNUAL retention." >> "$SCRIPT_LOG"
    fi
}

perform_daily_backup() {
    find "$WWW_DIR" -type f -path "*/inc/header.inc.php" | while read HEADER_FILE; do
        SITE_DIR=$(dirname $(dirname "$HEADER_FILE"))

        DB_NAME=$(grep "define('BX_DATABASE_NAME'" "$HEADER_FILE" | cut -d"'" -f4)
        DB_USER=$(grep "define('BX_DATABASE_USER'" "$HEADER_FILE" | cut -d"'" -f4)
        DB_PASS=$(grep "define('BX_DATABASE_PASS'" "$HEADER_FILE" | cut -d"'" -f4)
        DB_HOST=$(grep "define('BX_DATABASE_HOST'" "$HEADER_FILE" | cut -d"'" -f4)
        DB_SOCK=$(grep "define('BX_DATABASE_SOCK'" "$HEADER_FILE" | cut -d"'" -f4)
        SITE_URL=$(grep "define('BX_DOL_URL_ROOT'" "$HEADER_FILE" | grep -v 'isset' | cut -d"'" -f4 | sed 's|https\?://||;s|/||g')

        [ -z "$SITE_URL" ] && SITE_URL=$(basename "$SITE_DIR")

        echo "  Starting daily backup for $SITE_URL at $(date)" >> "$SCRIPT_LOG"

        TAR_FILE_PATH="$DAILY_DIR/html/${SITE_URL}-$DATE.tar.gz"
        tar -czf "$TAR_FILE_PATH" -C "$SITE_DIR" . 2>/dev/null
        if [ $? -eq 0 ]; then
            echo "  ✔️ Files backup for $SITE_URL completed." >> "$SCRIPT_LOG"
            link_backups "$SITE_URL" "$TAR_FILE_PATH" "html"
        else
            echo "  ❌ Files backup for $SITE_URL failed." >> "$SCRIPT_LOG"
        fi

        DB_FILE_PATH="$DAILY_DIR/db/${SITE_URL}.db-$DATE.sql.gz"
        mysqldump --single-transaction --quick --lock-tables=false \
                  --user="$DB_USER" --password="$DB_PASS" \
                  --host="$DB_HOST" --socket="$DB_SOCK" "$DB_NAME" | gzip > "$DB_FILE_PATH" 2>/dev/null
        if [ $? -eq 0 ]; then
            echo "  ✔️ Database backup for $SITE_URL completed." >> "$SCRIPT_LOG"
            link_backups "$SITE_URL" "$DB_FILE_PATH" "db"
        else
            echo "  ❌ Database backup for $SITE_URL failed." >> "$SCRIPT_LOG"
        fi
    done
}

# ==============================================================================
# 4. Execution & Cleanup
# ==============================================================================

perform_daily_backup

echo "--- Starting cleanup ---" >> "$SCRIPT_LOG"

find "$DAILY_DIR" -type f -mtime +$RETENTION_DAILY_DAYS -exec rm -f {} \;
echo "  🧹 Cleaned Daily backups older than $RETENTION_DAILY_DAYS days." >> "$SCRIPT_LOG"

find "$WEEKLY_DIR" -type f -mtime +$RETENTION_WEEKLY_DAYS -exec rm -f {} \;
echo "  🧹 Cleaned Weekly backups older than $RETENTION_WEEKLY_DAYS days." >> "$SCRIPT_LOG"

find "$MONTHLY_DIR" -type f -mtime +$RETENTION_MONTHLY_DAYS -exec rm -f {} \;
echo "  🧹 Cleaned Monthly backups older than $RETENTION_MONTHLY_DAYS days." >> "$SCRIPT_LOG"

echo "===== Backup rotation completed at $(date) =====" >> "$SCRIPT_LOG"
echo "" >> "$SCRIPT_LOG"
```

---

## ⏰ 4. Automate the Script with Cron

### Step A: Make Executable

```bash
chmod +x /opt/kabballa/scripts/daily_backup.sh
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
0 3 * * * /opt/kabballa/scripts/daily_backup.sh >>/dev/null
```

---

## 📧 5. Email Alert Behavior

| Type      | Behavior                                                           |
| --------- | ------------------------------------------------------------------ |
| ✅ Success | All logs written to `$SCRIPT_LOG`. No output → no email.           |
| ⚠️ Error  | Script writes to `stderr` → cron sends an alert email to `MAILTO`. |

---

## 🔍 6. Verification

After execution, check:

```bash
cat /opt/kabballa/data/backups/logs/backup_rotation.log
```

You should see:

```
===== Backup rotation completed at YYYY-MM-DD HH:MM:SS =====
```

---

## ✅ Summary

| Task                   | Command                                                   |
| ---------------------- | --------------------------------------------------------- |
| Make script executable | `chmod +x /opt/kabballa/scripts/daily_backup.sh`          |
| Test manually          | `/opt/kabballa/scripts/daily_backup.sh`                   |
| View logs              | `cat /opt/kabballa/data/backups/logs/backup_rotation.log` |
| Edit cron              | `crontab -e`                                              |
| Edit .env              | `nano /opt/kabballa/data/backups/.env`                    |

---

> 🧠 **Tip:** Extend the backup system with `rclone` or `rsync` to replicate backups to remote storage (e.g., S3, Google Drive, Ceph Object Gateway).

```
