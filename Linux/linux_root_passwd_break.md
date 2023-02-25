### Breaking Root Password

- Before system boot, **press e** to edit the boot menu
- Goto line ***rhgb quiet*** and replace it with the ***rd.break enforcing=0***
- After making the changes **press ctrl+x** to start the boot

**Note: *init=/bin/bash* has no safe reboot method, so avoid using in grub statement like we adding rd.break or enforcing=0**

After the boot is completed you will be prompted a shell and run the following command:
```bash
mount -o remount,rw /sysroot
chroot /sysroot
passwd root
press ctrl+d    # (logout)
mount -o remount,ro /sysroot
exit
# Now login with root and with the changed password
restorecon /etc/shadow
setenforce 1
getenforce
```
