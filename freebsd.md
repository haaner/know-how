# Boot / Partition Setup

```bash
$ gpart destroy ad0

#gpart create -s mbr ada0
$ gpart create -s gpt ad0
$ gpart add -t bios-boot -s 3M ada0

# Install bootloader in boot partition (not needed when booting with grub)
#gpart add -b 34 -s 512k -t freebsd-boot ada0
#gpart bootcode -p /boot/gptzfsboot -i 2 ada0

$ gpart add -t freebsd-zfs -l zroot ada0

# Full zpool backup and restore to a new pool
$ zpool import oldzroot
$ zfs snapshot -r oldzroot@movedata
$ zpool create zroot /dev/ada0s3
$ zfs send -vR oldzroot@movedata | zfs receive -vFd zroot
$ zfs destroy -r oldzroot@movedata

# IMPORTANT: This is needed so that zfsloader finds the right bootfs, otherwise it will will complain about missing "loader.lua"
$ zpool set bootfs=zroot/ROOT/default zroot 
```


# Grundlegende Zpool und ZFS - Kommandos

#### Alle Zpools verfügbar machen und die darin enthalten Datasets nicht automatisch mounten

    zpool import -N

#### Zpool umbenennen (und verfügbar machen)

    zpool import old-name new-name

#### Zpool persistieren und unzugänglich machen

    zpool export pool-name

#### Alle Datasets der verfügbaren Zpools (auf ihren vorgesehen Mountpoints) einhängen

    zfs mount -a

#### Alle gemounteten Datasets asuhängen

    zfs umount -a

#### Ein Dataset auf einem bestimmten Mountpoint einhängen

    mount -t zfs poolname/datasetname /mnt```

#### Den Mountpoint aller Datasets permanent anpassen

```bash
for dataset in $(zfs list -H -o name poolname); do
    zfs set mountpoint=/neuer/pfad/$dataset $dataset
done
```

#### Den Mountpoint für neue Datasets festlegen

    $ zfs set mountpoint=/neuer/pfad poolname


# ZFS Snapshots

```
$ zfs snapshot -r zroot@mysnapshot
$ zfs list -t snapshot

$ zfs rollback -r zroot@mysnapshot # für jeden zfs mount manuell ausführen
$ zfs destroy -r zroot@mysnapshot
```

# Manuell installierte Pakete

```
$ pkg prime-list
```

# pkg Aliase

```
$ pkg alias
```

# WLAN

```
$ ifconfig wlan0 create wlandev iwm0
$ ifconfig wlan0 scan
$ ifconfig wlan0 ssid "BirdWorX" # geht nur, wenn kein PW für Verbindung notwendig ist

$ cat < _EOF_ >> /etc/wpa_supplicant.conf
network={ 
    ssid="BirdWorX"
    psk="SecretPW"
    scan_ssid=1
}
_EOF_

$ wpa_supplicant -i wlan0 -c /etc/wpa_supplicant.conf
``````

# Update

```bash
$ freebsd-update fetch
$ freebsd-update install
```

# Upgrade 

```
$ freebsd-update -r 13.2-RELEASE upgrade
$ freebsd-update install
$ shutdown -r now
$ freebsd-update install
$ pkg-static upgrade -f
``````


# TODO:

## BSD-Boot: UEFI -> BIOS

And to be complete, if you have to reverse this change. Boot on the
disc1 of your FreeBSD version.

$ gpart delete -i 1 ada0
$ gpart add -t freebsd-boot -i 1 -s 512k ada0

It's important to specify a size of 512 KB since freebsd-boot cannot
exceed 545 KB (because of design of pmbr program in the protective MBR
of the GPT scheme). The following partition, freebsd-swap, is aligned to
1 MB, so there is a space of 1 MB available at the beginning of the
scheme (use gpart show to see that).

If your root file system is ZFS:
$ gpart bootcode -p /boot/gptzfsboot -i 1 ada0

Else if it is UFS:
$ gpart bootcode -p /boot/gptboot -i 1 ada0

Your system now can only boot under BIOS and not anymore with EFI.

[ https://forums.freebsd.org/threads/transfer-existing-drive-from-bios-motherboard-to-uefi-motherboard.68024/ ]

## BSD-Boot: BIOS -> UEFI 

You have to delete the freebsd-boot partition (assuming it's the first,
but if it's a default installation, it's actually the first):

$ gpart delete -i 1 ada0

Then, create an efi partition which must be a msdos file system.

$ gpart add -t efi ada0
$ newfs_msdos -F 12 -c 1 /dev/ada0p1

I selected a FAT12 (that belongs to the floppy world!) because there is
not enough room to build another FAT system. The default partition
freebsd-boot is 512 KB size.

You can now copy /boot/loader.efi:

$ mount -t msdosfs /dev/ada0p1 /mnt
$ mkdir -p /mnt/efi/boot
$ cp /boot/loader.efi /mnt/efi/boot/bootx64.efi
$ umount /mnt

(Most of these commands comes from: https://wiki.freebsd.org/UEFI)

---

https://wiki.freebsd.org/RootOnZFS/GPTZFSBoot
https://forums.freebsd.org/threads/booting-freebsd-via-grub.60422/

# vim: ts=4:sw=4
