# Cron Jobs & Bash Scripting Comprehensive Cheatsheet (Linux / DevOps)

This cheatsheet covers **cron scheduling + Bash scripting** from fundamentals to production usage. It is written with **DevOps, servers, CI/CD, and automation** in mind.

You’ll learn:

* How cron works internally
* Crontab syntax (all fields)
* Environment & permissions gotchas
* Writing robust Bash scripts
* Logging, error handling, notifications
* Real-world cron + Bash patterns

Official documentation links are included for deeper reference.

---

## PART A — CRON JOBS

---

## 1. What Cron Is

Cron is a **time-based job scheduler** in Unix/Linux systems.

It runs commands or scripts automatically at specified times.

Docs:

* [https://man7.org/linux/man-pages/man8/cron.8.html](https://man7.org/linux/man-pages/man8/cron.8.html)
* [https://man7.org/linux/man-pages/man5/crontab.5.html](https://man7.org/linux/man-pages/man5/crontab.5.html)

---

## 2. How Cron Works (Mental Model)

```text
cron daemon (crond)
   ↓
reads crontab files
   ↓
executes commands at scheduled times
   ↓
stdout / stderr → mail or logs
```

Important:

> Cron runs with a **minimal environment** (NOT your shell env).

---

## 3. Crontab Files

### User Crontab

```bash
crontab -e
```

### List Jobs

```bash
crontab -l
```

### Remove All Jobs

```bash
crontab -r
```

System crontabs:

* `/etc/crontab`
* `/etc/cron.d/*`

---

## 4. Crontab Syntax (VERY IMPORTANT)

```text
* * * * * command
│ │ │ │ │
│ │ │ │ └── Day of week (0–7, Sun=0 or 7)
│ │ │ └──── Month (1–12)
│ │ └────── Day of month (1–31)
│ └──────── Hour (0–23)
└────────── Minute (0–59)
```

---

## 5. Cron Field Values

| Symbol | Meaning         |
| ------ | --------------- |
| *      | every value     |
| */5    | every 5 units   |
| 1,3,5  | specific values |
| 1-5    | range           |

Example:

```cron
*/15 9-17 * * 1-5 /script.sh
```

→ Every 15 minutes, work hours, weekdays

---

## 6. Common Cron Examples

Run every day at midnight:

```cron
0 0 * * * /backup.sh
```

Run every reboot:

```cron
@reboot /startup.sh
```

Special strings:

```text
@hourly
@daily
@weekly
@monthly
```

---

## 7. Cron Environment Pitfalls

Cron:

* Does NOT load `.bashrc` or `.profile`
* Has limited `PATH`

### Always use:

* Absolute paths
* Explicit environment variables

Example:

```cron
PATH=/usr/bin:/bin
0 * * * * /usr/bin/python3 /app/script.py
```

---

## 8. Logging Cron Jobs

```cron
* * * * * /script.sh >> /var/log/script.log 2>&1
```

* `>` overwrite
* `>>` append
* `2>&1` redirect stderr

---

## 9. Debugging Cron

Check logs:

```bash
/var/log/syslog
/var/log/cron
journalctl -u cron
```

Test manually:

```bash
bash script.sh
```

---

## PART B — BASH SCRIPTING

---

## 10. Bash Script Basics

### Shebang

```bash
#!/usr/bin/env bash
```

Make executable:

```bash
chmod +x script.sh
```

Run:

```bash
./script.sh
```

Docs: [https://www.gnu.org/software/bash/manual/bash.html](https://www.gnu.org/software/bash/manual/bash.html)

---

## 11. Variables

```bash
NAME="world"
echo "Hello $NAME"
```

Read input:

```bash
read -p "Enter name: " NAME
```

---

## 12. Arguments

```bash
echo "First arg: $1"
echo "All args: $@"
echo "Arg count: $#"
```

---

## 13. Exit Codes

```bash
command
if [ $? -ne 0 ]; then
  echo "Failed"
  exit 1
fi
```

Short form:

```bash
command || exit 1
```

---

## 14. Conditionals

```bash
if [ "$ENV" = "prod" ]; then
  echo "Production"
elif [ "$ENV" = "dev" ]; then
  echo "Development"
else
  echo "Unknown"
fi
```

Test operators:

* `-f` file exists
* `-d` directory exists
* `-z` empty string

---

## 15. Loops

### For Loop

```bash
for i in 1 2 3; do
  echo $i
done
```

### While Loop

```bash
while read line; do
  echo "$line"
done < file.txt
```

---

## 16. Functions

```bash
backup() {
  echo "Backing up"
}

backup
```

---

## 17. Command Substitution

```bash
DATE=$(date +%F)
echo "$DATE"
```

---

## 18. Error Handling (Production Grade)

```bash
set -euo pipefail
```

Meaning:

* `-e` exit on error
* `-u` undefined vars fail
* `-o pipefail` pipeline fails if any command fails

---

## 19. Logging in Bash

```bash
log() {
  echo "$(date '+%F %T') $1" >> app.log
}

log "Started"
```

---

## 20. Environment Files (.env)

```bash
export $(grep -v '^#' .env | xargs)
```

---

## 21. Bash + Cron (Best Pattern)

### Script

```bash
#!/usr/bin/env bash
set -euo pipefail

source /etc/profile

/app/backup.sh >> /var/log/backup.log 2>&1
```

### Crontab

```cron
0 2 * * * /app/run-backup.sh
```

---

## 22. Notifications from Bash

### Email

```bash
echo "Job finished" | mail -s "Cron Status" admin@example.com
```

### Slack (Webhook)

```bash
curl -X POST -H 'Content-type: application/json' \
--data '{"text":"Job completed"}' $SLACK_WEBHOOK
```

---

## 23. Security Best Practices

* Never hardcode secrets
* Restrict script permissions
* Validate inputs
* Use least privilege

---

## 24. Typical Real-World Use Cases

* Database backups
* Log rotation
* Docker cleanup
* Certificate renewal
* Monitoring & health checks

---

## 25. Official References (Bookmark)

* Cron man page: [https://man7.org/linux/man-pages/man5/crontab.5.html](https://man7.org/linux/man-pages/man5/crontab.5.html)
* Bash manual: [https://www.gnu.org/software/bash/manual/](https://www.gnu.org/software/bash/manual/)
* Advanced Bash Guide: [https://tldp.org/LDP/abs/html/](https://tldp.org/LDP/abs/html/)

---

## 26. How This Fits Your Stack

```text
Cron → Bash → Docker → GitHub Actions → Terraform → AWS
```

Cron handles **time-based automation** where CI/CD is event-based.

---

## 27. Next Advanced Topics

* systemd timers vs cron
* Robust lockfiles (flock)
* Distributed cron jobs
* Observability for cron tasks

(Ask anytime for production templates or deep dives.)
