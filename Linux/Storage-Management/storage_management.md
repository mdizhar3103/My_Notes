## Storage Management
```bash
lsblk
fdisk -l /dev/device_name       # list partition in a device
blkid /dev/device_name          # to get block device details

# create partition - refer to online doc
# Confgure partition to specific filesystem
# Extend Partition
# Create LVM, VG
# Configure NFS server
# Configure AutoFS server
```

## Swap Management
```bash
swapon --show
# create swap, lets check if swaps are available or not
lsblk

mkswap /dev/partition_name
swaop -v /dev/created_swap_partition_name
swapon --show
swapoff /swap_name      # to turnoff swap

# creating swap from file
dd if=/dev/zero of=/swap bs=1M count128         
# using dd command to generate swap file
# /dev/zero if special utility that generates zeros infinite number of times.
# /swap where the zeros are writtern
# bs=1M write 1 megabyte of data 128 times

dd if=/dev/zero of=/swap bs=1M count128 status=progress     # to check the status
chmod 600 /swap
mkswap   /swap
swapon -v /swap
swapon -s
```

## Mount commands
```bash
findmnt
findmnt -t xfs      # list only type xfs
findmnt -t xfs,ext4 

# mounting using read-only
mount -o ro /dev/partition_name /mount_folder_path

# unmount the device
umount /folder_path

# mounting with no uuid
mount -o nouuid /dev/partition_name /mount_folder_path

# mount with no program to execute on the device and no suid
mount -o ro,noexec,nosuid /dev/partition_name /mount_folder_path
findmnt -t xfs,ext4 

# remounting the mounted the device with read-write option 
mount -o remount,rw,noexec,nosuid /dev/partition_name /mount_folder_path

mount -o allocsize=32K /dev/partition_name /mount_folder_path
# make entry in fstab to make it permanent
```
> Note: Only mounted paths are shown

