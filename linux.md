### LVM

#### Snapshot erstellen, wiederherstellen bzw. löschen

$ lvcreate  -L360G -s -n 150-disk-snap /dev/ubuntu-vg/server150-disk  
$ lvconvert --merge /dev/ubuntu-vg/150-disk-snap  
$ lvremove /dev/ubuntu-vg/150-disk-snap

#### Falls eine Snapshot-Wiederherstellung fehlschlägt 

Den Server bzw. Prozess herunterfahren der die ursprüngliche LV blockiert.

$ lvchange --refresh ubuntu-vg  
$ lvs 

Sobald der Data%-Wert auf 0 heruntergezählt hat, ist der Snapshot gemerged.