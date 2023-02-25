# Scheduling Jobs
```bash
cat /etc/crontab
tail -f /var/log/cron
crontab -e                      # edit cron job
crontab -l                      # list cron jobs
crontab -e -u <username>        # edit cron job of specific user
crontab -r                      # to remove cron jobs
crontab -r -u <username>        # to remove cron jobs of specific user

# scheduling jobs in specific directory
ls /etc/cron.daily/
ls /etc/cron.hourly/
ls /etc/cron.monthly/
ls /etc/cron.weekly/
```
