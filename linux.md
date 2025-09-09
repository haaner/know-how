# apt / dpkg 

#### Installierte Pakete der Größe nach sortieren auflisten
		
	dpkg-query -Wf '${Installed-Size}\t${Package}\n' | sort -n
		
#### Installierte (Alt-)Kernel auflisten
	 
	dpkg --list | grep -i -E --color 'linux-image|linux-kernel' | grep '^ii'

#### Alt-Kernel entfernen

	apt purge linux-image-$i-generic linux-headers-$i-generic linux-modules-$i-generic
		
#### Kernel reinstallieren
	 
	apt-get install --reinstall linux-image-generic linux-image-5.15.0-88-generic
		
#### Separates Repository hinzufügen
	
	add-apt-repository ppa:rdiff-backup/rdiff-backup-backports
	add-apt-repository ppa:ondrej/php	
	
#### Repository-Key hinzufügen
	
	apt-key adv --keyserver keyserver.ubuntu.com --recv 3B4FE6ACC0B21F32
	apt-key export 3B4FE6ACC0B21F32 | gpg --dearmour -o key.gpg
	apt-key del 3B4FE6ACC0B21F32
	mv key.gpg /etc/apt/trusted.gpg.d
				
#### Nur Security-Updates einspielen
	
	apt-get -s dist-upgrade | grep "^Inst" | grep -i securi | awk -F " " {'print $2'} | xargs apt-get install

#### Zurückgehaltene Pakete einspielen

	apt dist-upgrade # falls dies nicht hilft:
	apt install --only-upgrade failing-package # falls dies nicht hilft:
	apt --with-new-pkgs upgrade failing-package

#### Dritt-Repository in Unattended Upgrades einbeziehen
	
Nach Ausführung von `apt policy`:

	...
 	500 https://ppa.launchpadcontent.net/ondrej/php/ubuntu jammy/main i386 Packages
     release v=22.04,o=LP-PPA-ondrej-php,a=jammy,n=jammy,l=PPA for PHP,c=main,b=i386
     origin ppa.launchpadcontent.net
	...

die Werte von o= und a= in der Datei /etc/apt/apt.conf.d/50unattended-upgrades innerhalb von Unattended-Upgrade::Allowed-Origins folgendermaßen hinzufügen:

   	"LP-PPA-ondrej-php:${distro_codename}";
				
#### Manuell installierte Pakete anzeigen

	apt-mark showmanual		

#### Paket-Selektion erstellen

	dpkg --get-selections > packages.lst
	
#### Paket-Selektion einspielen

	apt-cache dumpavail | dpkg --merge-avail
	dpkg --set-selections < packages.lst
	apt install dselect ; apt dselect-upgrade

#### Konfigurationen cleanen:
	
	find /etc \( -iname '*-' -o -iname '*~' -o -iname '*.bak' -o -iname '*.old' -o -iname '*.ucf-*' -o -iname '*.dpkg-*' -o -iname '*.distUpgrade' \) -exec rm '{}' +

# Disk 

#### Block-Devices inklusive Größen und RAID-Gruppierung auflisten
		
	lsblk
		
#### UUIDs auflisten
	
	lsblk -lf
		
# MD

#### RAID1 anlegen
	
	mdadm --create --verbose /dev/md0 --level=1 --raid-devices=2 /dev/sda3 /dev/sdc3
		
#### Platte aus RAID-Verbund entfernen
	
	mdadm --manage /dev/md0 --fail /dev/sda1
	mdadm --manage /dev/md0 --remove /dev/sda1
		
#### Platte in RAID-Verbund aufnehmen
	
	mdadm --manage /dev/md0 --add /dev/sda1
		
#### RAID löschen
	
	mdadm --stop /dev/md1 
	mdadm --remove /dev/md1
	mdadm --zero-superblock /dev/sdb1 /dev/sdd1 # wichtig sonst gibts evtl. Probleme beim Booting (mit grub)
		
#### RAID wieder starten (um zuvor gestoppte Verbünde wieder zu starten)
	
	mdadm --assemble --scan	
		
#### Platte aus Verbund nehmen und durch neue Platte ersetzen
		
	mdadm --manage /dev/md0 --fail /dev/sda1
	mdadm --manage /dev/md1 --fail /dev/sda2
	mdadm /dev/md0 -r /dev/sda1
	mdadm /dev/md1 -r /dev/sda2
	grub-install /dev/sdb
	
	halt
		
	sfdisk -d /dev/sdb | sfdisk /dev/sda
	mdadm /dev/md0 -a /dev/sda1
	mdadm /dev/md1 -a /dev/sda2
	grub-mkdevicemap -n
	grub-install /dev/sda

# LVM

#### Snapshot erstellen, wiederherstellen bzw. löschen:
	
	lvcreate  -L360G -s -n 150-disk-snap /dev/ubuntu-vg/server150-disk
	lvconvert --merge /dev/ubuntu-vg/150-disk-snap
	lvs -a # zeigt an wie lange die endgültige Wiederherstellung noch dauert
	lvremove /dev/ubuntu-vg/150-disk-snap

#### Falls eine Snapshot-Wiederherstellung fehlschlägt:
     
Den Server bzw. Prozess herunterfahren, der die ursprüngliche LV blockiert
     
	lvchange --refresh ubuntu-vg
	lvs

Sobald der Data%-Wert auf 0 heruntergezählt hat, ist der Snapshot gemerged.

#### LV klonen
	
	lvconvert --type mirror --alloc anywhere -m1 /dev/rootvg/test
	lvs -a -o +devices | egrep "LV|test"

> LV              VG     Attr       LSize .. ove Log       Cpy%Sync Convert Devices                          
> test            rootvg mwi-aom--- 1.00g      [test_mlog] 100.00           test_mimage_0(0),test_mimage_1(0)
> [test_mimage_0] rootvg iwi-aom--- 1.00g                                   /dev/sda5(87200)                 
> [test_mimage_1] rootvg iwi-aom--- 1.00g                                   /dev/sda5(36009)                 
> [test_mlog]     rootvg lwa-aom--- 4.00m                                   /dev/sda5(66575)                 

Check that Cpy%Sync is 100% finished; both copies are in sync then. Now let's break the mirror !

	lvconvert --splitmirrors 1 --name testCopy /dev/rootvg/test
	mount /dev/rootvg/testCopy /mnt/test_copy
	
#### LV Binärkopie erstellen
		
	lvcreate --snapshot --name <the-name-of-the-snapshot> --size <size> /dev/volume-group/logical-volume
	lvcreate --name <logical-volume-name> --size <size> the-new-volume-group-name
	dd if=/dev/volume-group/snapshot-name of=/dev/new-volume-group/new-logical-volume


# LUKS

#### Verschlüsselte Partition anlegen und beim Login mounten
	
	lvcreate -L +512M kali-vg -n safe
	cryptsetup luksFormat /dev/kali-vg/safe
	cryptsetup luksOpen /dev/kali-vg/safe safe
	mkfs.ext4 /dev/mapper/safe
	mkdir Safe
	mount /dev/mapper/safe Safe
	touch Safe/test
	umount Safe
	cryptsetup luksClose safe

In /etc/security/pam_mount.conf.xml folgende Zeile einfügen:

	<volume user="heiko" fstype="crypt" path="/dev/kali-vg/safe" mountpoint="~/Safe" options="noatime,noexec,nodev,nosuid" />


# systemctl

#### Aktivierte Services auflisten

	systemctl list-unit-files | grep enabled | sort

#### Service aktivieren

	systemctl enable php7.4-fpm-eibe

#### Service restarten

	systemctl restart apache2 

# UFW

#### Regeln auflisten

	ufw status numbered
		
#### Ports freigeben
		
	ufw allow proto tcp from 88.99.12.250 to any port 22,5666 comment "Hetzner Nagios(neu)"