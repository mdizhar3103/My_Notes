# Resource Management
```bash
df -h
du -sh <folder_name>
free -h
uptime
    # load average: last_min avg, last_5_min avg, last_15_min_avg
lscpu
lspci
xfs_repair -v /dev/partition_name
fsck.ext4 -vfp /dev/partition_name
    # f: force
    # v: verbose
    # p: prompt for yes or no

systemctl list-dependencies
```

### Troubleshooting CPU & Memory
```bash
yum install sysstat
systemctl start sysstat
systemctl enable sysstat
systemctl status sysstat

# check default cron entry for sysstat (which run every 10 mins as default)
cat /etc/cron.d/sysstat

vim /etc/default/sysstat
    # change ENABLED to `true` if set to `false`

man procfs

# High CPU Utilization
mpstat <Interval> <Seconds>
    # mspstat 1 60
    # Run every after 1 second for 60 seconds
    
mpstat  -P <PID> <Interval> <Seconds>
    # mpstat -P 0 1 60

sar -u 
```
