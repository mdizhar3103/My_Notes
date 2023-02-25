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
