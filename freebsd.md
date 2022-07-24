# Boot and zpool partition setup


```bash
gpart destroy ad0

#gpart create -s mbr ada0
gpart create -s gpt ad0
gpart add -t bios-boot -s 3M ada0

# Install bootloader in boot partition (not needed when booting with grub)
#gpart add -b 34 -s 512k -t freebsd-boot ada0
#gpart bootcode -p /boot/gptzfsboot -i 2 ada0

gpart add -t freebsd-zfs -l zroot ada0

# Full zpool backup and restore to a new pool
zpool import oldzroot
zfs snapshot -r oldzroot@movedata
zpool create zroot /dev/ada0s3
zfs send -vR oldzroot@movedata | zfs receive -vFd zroot
zfs destroy -r oldzroot@movedata

# IMPORTANT: This is needed so that zfsloader finds the right bootfs, otherwise it will will complain about missing "loader.lua"
zpool set bootfs=zroot/ROOT/default zroot 
```
# ZFS Snapshots

zfs snapshot -r zroot@mysnapshot
zfs list -t snapshot

# Manuell installierte Pakete

pkg prime-list

# pkg Aliase

pkg alias

---

https://wiki.freebsd.org/RootOnZFS/GPTZFSBoot
https://forums.freebsd.org/threads/booting-freebsd-via-grub.60422/

# vim: ts=4:sw=4
