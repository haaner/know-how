# Apache 

## mod_rewrite

#### Using variables on the right side of RewriteCond by back-reference: \1

	RewriteCond %{DOCUMENT_ROOT},%{REQUEST_FILENAME} ^([^,]*+),\1/*+(.*?/)/*+([^/]*)$ 
	RewriteRule ^ - [E=CURPATH:%2,E=CURNAME:%3]
	
[ http://www.ckollars.org/apache-rewrite-htaccess.html#precedence ]
		
		
# Composer

#### ext-gmp ignorieren (erst ab Composer 2 möglich)
	
	php bin/composer.phar install --ignore-platform-req=ext-gmp
		
#### Content-Hash in composer.lock korrigieren: 
		
	composer update --lock


# [FreeBSD](freebsd.md)		


# GDB

Via `--args` können Kommandozeilen-Parameter an gdb übergeben werden

	gdb --args ./test -p1 --param=2 

#### Executable einlesen

	file ./test

#### Sourcen "refreshen"

	directory

#### FAIL-Funktion aus error.cpp auflisten

	list error.cpp:FAIL

#### Zeile 10 aus error.cpp auflisten
		
	list error.cpp:10
	
#### Breakpoint auf Zeile 12 setzen

	b 12
	b 12 if stats.iters <= 25
		
#### Breapoints listen

	i b

#### Nachträgliche Bedingung für BP ergänzen

	cond 1 i == 99

#### Breakpoint disablen / enablen / löschen

	dis 1 / en 1 / d 1

#### Breakpoints speichern und wieder einlesen

	save breakpoints file.txt
	source file.txt

#### Watchpoint setzen
	
	watch stats.iters

#### Variable bei jedem Stop anzeigen
		
	display stats.iters

#### Starten / Fortsetzen / Steppen / Nächste
		
	r / c / s / n

#### Funktion beenden

	fin

#### Stack hoch / runter

	up / down

#### Backtrace anzeigen / 3. Frame selektieren

	bt
	frame 3

	info frame
	info args
	info locals

#### Funktion aufrufen (evtl. vorher: set overload-resolution off)
		
	p member_function()

		
# Grub

- Installation (bei RAIDs auf alle beteiligte Platten klatschen, die die boot-Partition enthalten) 

	```	
		grub-install /dev/sda
		grub-install /dev/sdb 
	```
	
	Auszug aus History des internen Backup-Servers
	```	
	1502  cfdisk /dev/sdb
	1503  mdadm --manage /dev/md0  --add /dev/sdb2
	1504  mdadm --manage /dev/md1  --add /dev/sdb4
	1505  mdadm --examine --scan
	1506  vi /boot/grub/grub.cfg
	1507  update-grub
	1508  update-initramfs -u
	1509  grub-install /dev/sda
	1510  grub-install /dev/sdb
	1511  pstree
	1512  reboot
	```

## InfluxDB

   - Datenpunkte hinzufügen via API

```
	for i in $(seq 1 100) ; do 
		curl -XPOST "http://localhost:8086/api/v2/write?org=intercorp&bucket=corpus" \
			--header "Authorization: Token t1kt0k" \
			--data-raw "heiko,tag1=194,tag2=5755 field1=$RANDOM" 
		sleep 5
	done
```

## LFTP
		
- Remote Verzeichnis komplett downloaden

	```lftp> mirror .```		
- Lokales Verzeichnis hochladen und GIT-Ordner weglassen

	```lftp> mirror -R . --exclude .git/```

- Verbindung via SFTP
	
	```lftp sftp://user@host```
		
	(Bei Fehlermledung wg. 'Host key verification' zuvor kurz mit SSH verbinden)			
		
## macOS

### Home/End Key-Binding korrigieren

```
mkdir -p $HOME/Library/KeyBindings

echo '{
	/* Remap Home / End keys to be correct */
	"\UF729" = "moveToBeginningOfLine:"; /* Home */
	"\UF72B" = "moveToEndOfLine:"; /* End */
	"$\UF729" = "moveToBeginningOfLineAndModifySelection:"; /* Shift + Home */
	"$\UF72B" = "moveToEndOfLineAndModifySelection:"; /* Shift + End */
	"^\UF729" = "moveToBeginningOfDocument:"; /* Ctrl + Home */
	"^\UF72B" = "moveToEndOfDocument:"; /* Ctrl + End */
	"$^\UF729" = "moveToBeginningOfDocumentAndModifySelection:"; /* Shift + Ctrl + Home */
	"$^\UF72B" = "moveToEndOfDocumentAndModifySelection:"; /* Shift + Ctrl + End */
}' > $HOME/Library/KeyBindings/DefaultKeyBinding.dict
```

MySQL:

   - root-Passwort in Datei speichern und verwenden
   	
		$ cat < _END_ >> .my.cnf
		
		[client]
		user=root
		password=MyPassword
			
		_END_
	
		$ chmod 600 .my.cnf
		$ mysql --defaults-extra-file=~/.my.cnf
		
	- root-Passwort ändern

		ALTER USER 'root'@'localhost' IDENTIFIED WITH caching_sha2_password BY 'yourpasswd';

	- Alle Tabellen und den zugeh�rigen LOCK-Status anzeigen:
	
		SHOW OPEN TABLES

	- Alle gelockten Tabellen anzeigen:
    	
		SHOW OPEN TABLES WHERE In_use > 0		
		
	-  Let's see the list of the current processes, one of them is locking your table(s)

		SHOW PROCESSLIST;

	- Kill one of these processes

		KILL <put_process_id_here>;
		
	- Keys f�r Abfrage ermitteln (f�r die Optimierung von Zugriffszeiten):
	
		EXPLAIN SELECT SQL_NO_CACHE id, COUNT(*) FROM semmel.Person WHERE tmpName LIKE 'horst%' 
		
	- Summierung �ber gruppierten Sub-Select:
	
		SELECT SUM(sub.totals) FROM (SELECT COUNT(*) AS totals FROM `MetaField` GROUP BY srcId) AS sub

	- Tabellen / Daten aus Bin�rdateien wiederherstellen:
		
		Wenn es sich um eine MyISAM-Tabelle handelt:

		- Die Dateien *.frm, *.MYD, *.MYI in ein Datenbank-Verzeichnis "my_restore" unterhalb von /var/lib/mysql kopieren.

		# cp myTable.* /var/lib/mysql/my_restore
		# chown -R mysql:mysql /var/lib/mysql/my_restore

		- Den MySQL-Serverprozess reloaden bzw neu-starten:

		# /etc/init.d/mysql reload

		---

		Wenn sich um eine INNODB-Tabelle handelt:

		- Die urspr�nglichen CREATE-Befehle neu generieren lassen:

		# mysqlfrm --diagnostic myTable.frm > myTable.sql

		- Das Row-Format der wiederherzustellenden-Tabelle ermitteln und gg.falls den CREATE-Befehl anpassen, z.B.

		CREATE TABLE my_restore.myTable ... ENGINE=Innodb ROW_FORMAT=Compact

		- Mit MySQL-Server verbinden und das sql-File sourcen und danach die implizit erzeugte ibd-Datei discarden:

		mysql> SOURCE myTable.sql;
		mysql> ALTER TABLE myTable DISCARD TABLESPACE;

		- Das wiederherzustellende ibd-File in das Datenbank-Verzeichnis
		  "my_restore" kopieren und auf dem MySQL-Server folgenden Befehl ausf�hren:

		# cp myTable.ibd /var/lib/mysql/my_restore
		# chown -R mysql:mysql /var/lib/mysql/my_restore

		mysql> ALTER TABLE myTable IMPORT TABLESPACE;

		- Importierte Daten begutachten:

		mysql> SELECT * FROM myTable;
		
Jquery: 
	- Attribut-basierte Selektionen:
		$('input[type="radio"][id^="DealCostPosition"][id*="_provideType"]').each(function() {});
		
Excel:
	- Teilbereiche summieren:

		=SUMMEWENN(A2:A10;"Bezahlt";B2:B10) 
		=SUMME( -- die strg-taste gedr�ckt halten und alle zellen markieren, die addiert werden sollen - Klammer zu und RETURN dr�cken
		=SUMME(B1:B4;B6:B8) (Bereiche definieren)		
		
# PDF
	
#### Kompatibilitäts-Level anpassen
	
	gs -sDEVICE=pdfwrite -dCompatibilityLevel=1.3 -o output.pdf input.pdf	

#### Kompression entfernen

	gs -sDEVICE=pdfwrite -dPDFSETTINGS=/prepress -o output.pdf input.pdf	

Rsync:
		
	- Server kopieren
	
		rsync -avz --exclude '/proc' --exclude '/sys' --exclude '/dev' --exclude '/boot' --exclude '/etc/fstab' --exclude '/etc/network' --exclude '/etc/netplan' / 88.99.12.250:/
		
Screen:
		Ctrl+a S - Split screen	(horizontally)
		Ctrl+a | - Split screen (vertically)
			
		Ctrl+a TAB - Change Screen Window
		Ctr+a ESC - Copy / paste mode => SPACE - Set mark
		
		http://www.pixelbeat.org/lkdb/screen.html

SCSS: 

	- Per Kommandozeile SCSS-Dateien kompilieren:
	
		gem-sass Command-Line: 
			c:\Ruby23-x64\bin\scss.bat
				--style compact --unix-newlines --update $FileName$:../css/$FileNameWithoutExtension$.css

		node-sass CommandLine: 
			node c:\Users\User\AppData\Roaming\npm\node_modules\node-sass\bin\node-sass
				--output-style compact --output ../css --source-map true --source-map-contents $FileName$		
		
SVN:
	To ignore all files with the .jpg extension, use:
		svn propset svn:ignore "*.jpg" .

	If you want to ignore multiple files or folders, use:
		svn propedit svn:ignore .
		
	Resolve tree conflict:
		svn resolve --accept working -R <path>
		
	Resolve file conflict:
		svn resolved <file>
		 
		svn resolve --accept mine-full <file>
		 
	Remove file from version control but keep it locally:
		svn rm --keep-local <file>

	Add only directory but no contents:
		svn add --depth=empty <dir>

SSL:
	RSA Private/Public-Key erzeugen:
		openssl genrsa -out privkey.pem 2048
		openssl rsa -pubout -in privkey.pem -out pubkey.pem
	
	Entfernen der Passphrase aus einem privaten Schl�ssel
		openssl rsa -in privateKey.pem -out newPrivateKey.pem
	
	Zertifikate ausgeben:
		openssl s_client -connect mail.intercorp.de:465
		openssl s_client -connect mail.intercorp.de:25 -starttls smtp
		openssl s_client -connect mail.intercorp.de:587 -starttls smtp

	Ablaufdatum des Zertifikats ausgeben:
		echo "" | openssl s_client -servername staub-designlight.ch -connect shopware.intercorp.de:443 | openssl x509 -dates -noout

	Zertifikate konvertieren: (siehe auch https://www.hasslinger.com/index.php/en/blog/ssl-zertifikate-mit-openssl-konvertieren)
		openssl x509 -inform der -in lffepsprd1-rz-sued-bayern-de.crt -out certificate.pem
		openssl pkcs7 -print_certs -in certificate.p7b -out certificate.pem
		
	Generieren eines selbstsigniertes Zertifikats
		openssl req -x509 -sha256 -nodes -days 365 -newkey rsa: 2048 -keyout privateKey.key -out certificate.crt
	
	Generieren eines neuen privaten Schl�ssels und eine neue Zertifikatsignierungsanforderung
		openssl req -out CSR.csr -new -newkey rsa: 2048 -nodes -keyout privateKey.key
		
	Generieren einer Zertifikatsignierungsanforderung (Certificate Signing Request, CSR) f�r einen vorhandenen privaten Schl�ssel
		openssl req -out CSR.csr -key privateKey.key -new

	Generieren einer Zertifikatsignierungsanforderung basierend auf einem vorhandenen Zertifikat
		openssl x509 -x509toreq -in certificate.crt -out CSR.csr -signkey privateKey.key
		
	LetsEncrypt-Zertifikat ausstellen:
		 letsencrypt certonly --webroot -w
/var/www/vhosts/staub-designlight.ch/letsencrypt/ -d dev.staub-designlight.ch
-d www.staub-designlight.ch -d staub-designlight.ch -d
www.staub-designlight.com -d staub-designlight.com  -d
www.staub-designlight.de -d staub-designlight.de -d www.staub-designlight.fr
-d staub-designlight.fr -d www.staub-dl.ch -d staub-dl.ch	

HTTPS auf HTTP umsetzen:
	
	$ socat OPENSSL-LISTEN:443,cert=cert.pem,key=key.pem,verify=0,fork,reuseaddr TCP4-CONNECT:localhost:80
	
UNIX-Terminal:

	- Anzahl der maximal anzeigbaren Spalten anpassen
		
		$ stty -a | grep columns
		$ stty columns 120
	  
VIMdiff:

	$ vimdiff /etc/orig /etc/new
	
	]c :	next difference
	[c :	previous difference
	do 		Gg.�berliegende Seite �bernehmen (obtain)
	dp 		Momentane Seite behalten (put)
	zo 		open folded text
	zc 		close folded text
	Ctrl-W W toggle between diff columns
	
VIM:

	- Beim Pasting keine Text-Formatierung durchf�hren
		:set paste
	
VirtualBox:

	- HardDrive <-> VDI klonen

	$ VBoxManage convertfromraw /dev/sda MyImage.vdi --format VDI
	$ VBoxManage clonehd --format RAW file.vdi | dd of=/dev/sda
	
Xen:

	xl list 
	xl destroy 
	xl create /etc/xen/win2008r2
	
Windows:

	Pakete entfernen (via PowerShell), beispielsweise WindowsStore:
		
		Get-AppxPackage *windowsstore* | Remove-AppxPackage


