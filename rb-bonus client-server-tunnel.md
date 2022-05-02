# Installation und Aktivierung verschlüsselter Tunnel

Pendenzen allgemein

1. 

## Generell für alle Varianten

Neuer User `admin` anlegen 
https://raspibolt.org/guide/raspberry-pi/system-configuration.html#add-the-admin-user-and-log-in-with-it

Verzeichnis `/data`erstellen

```sh
admin@rbbs~$ sudo mkdir /data
```

Optional falls ssh-Zugriff auf RaspiBolt notwendig ist.

-  Schlüssel generieren (`/home/lnd/.ssh/id_rsa`) und auf `admin@raspibolt/home/asdmin/.ssh/authorized_keys` hinzufügen

```sh
ssh-keygen -t rsa -b 4096
```





## Variante sshuttle

https://sshuttle.readthedocs.io/en/stable/installation.html

```ssh
$ sudo apt install sshuttle
```



Login auf RaspiBolt mit SSH-Keys konfigurieren
https://raspibolt.org/guide/raspberry-pi/security.html#login-with-ssh-keys

Test Login mit SSH-Key

```ssh
$ ssh admin@raspibolt
```

Beim ersten Zugriff wird der Fingerprint zur Überprüfung angezeigt.

sshuttle verbinden
$ sshuttle -r admin@raspibolt 0.0.0.0/0

Mit Ctrl C abbrechen

Pendenzen:
1. ev. noch sshuttle --sudoers ausführen. siehe: https://sshuttle.readthedocs.io/en/stable/manpage.html#cmdoption-sshuttle-sudoers
2. mit nfs ein Laufwerk zu mounten

## Variante sshfs
https://linuxize.com/post/how-to-use-sshfs-to-mount-remote-directories-over-ssh

https://www.digitalocean.com/community/tutorials/how-to-use-sshfs-to-mount-remote-file-systems-over-ssh

### Installing SSHFS

SSHFS is available from the default Debian repositories. Update the packages index and install the sshfs client by typing:

```sh
admin@rbbs~$ sudo apt update
admin@rbbs~$ sudo apt install sshfs
```

Kontrolle ob sshfs -Module im Kernel geladen ist

```sh
admin@rbbs~$ lsmod | grep fuse
```

```
> fuse                  167936  1
```



Analog RaspiBolt wird ein Benutzer lnd und das Datendirectory /data/lnd hinzugefügt

- Auf dieses lokale Directory /data/lnd wird später das RaspiBolt-Directory /data/lnd gemountet

```sh 
admin@rbbs~$ sudo adduser --disabled-password --gecos "" lnd
admin@rbbs~$ sudo mkdir /data/lnd
admin@rbbs~$ sudo chown -R lnd:lnd /data/lnd
```

- Pendenz: Falls Tor installiert wird, muss lnd eventuell noch zur Gruppe debian-tor hinzugefügt werden
  Pendenz: Danicht mit dem Benutzer lnd auf rb-bonus gearbeitet wird, ist diese Gruppenzuordnung voraussichtlich nicht nötig
  Pendenz: admin@rbbs~$  sudo usermod -a -G debian-tor lnd

- Add the user “admin” to the group “lnd”

```sh
admin@rbbs~$ sudo adduser admin lnd
```



### lnd Key exchange

Mit User ``lnd`` Key generieren, damit sich dieser Benutzer beim Raspibolt für den mount des Laufwerks anmelden kann.

-  Schlüssel generieren (`/home/lnd/.ssh/id_rsa`) und auf `admin@raspibolt/home/asdmin/.ssh/authorized_keys` hinzufügen

```sh
admin@rbbs~$ sudo su - lnd
lnd@rbbs~$ ssh-keygen -t rsa -b 4096
lnd@rbbs:~$ cat .ssh/id_rsa.pub
```

- angezeigter Public-Key von `` lnd@rbbs`` kopieren.
- Auf admin@raspibolt den Benutzer auf lnd wechseln und die Datei ``authorized_keys``erstellen und den pulic-key von lnd@rbbs einfügen

```sh
admin@raspibolt:~$ sudo su - lnd
lnd@raspibolt:~ $ nano .ssh/authorized_keys
```

### Manual Mount the remote lnd Directory

Das  remote directory mit dem sshfs Befehl verbinden (-o debug zum testen):

```sh
lnd@rbbs~$ sshfs -o debug lnd@raspibolt:/data/lnd /data/lnd
```

Auf einem zweiten Terminal-Fenster mit ``admin@rbbs`` anmelden und auf ```lnd```wechseln

In Verzeichnis ``/data/lnd`` wechseln und Dateien auflisten. Im ersten Fenster sind die Aktivitäten aufgrund der Option ``-o debug`` zu beobachten

```sh
lnd@rbbs:/data/lnd$ cd /data/lnd
lnd@rbbs:/data/lnd$ ls -al
```

Im ersten Terminal-Fenster sind die Aktivitäten aufgrund der Option ``-o debug`` zu beobachten

```
[00026] LSTAT
  [00026]          ATTRS       41bytes (4ms)
[00027] OPENDIR
  [00027]         HANDLE       17bytes (3ms)
[00028] READDIR
[00029] READDIR
  [00028]           NAME     1507bytes (4ms)
  [00029]         STATUS       32bytes (4ms)
[00030] READDIR
[00031] READDIR
[00032] CLOSE
  [00030]         STATUS       32bytes (4ms)
  [00031]         STATUS       32bytes (4ms)
  [00032]         STATUS       28bytes (3ms)
```

### Permanent Mount the remote lnd Directory

Das Verzeichnis @raspibolt/data/lnd soll automatisch beim start von rbbs verbunden werden. Das Verzeichnis soll bei einem Unterbruch wieder Verbunden werden. Die Zugriffsrechte sollen analog dem Verzeichnis @raspibolt/data/lnd gelten.

Für den Mount wird die Datei /etc/fstab ergänzt:

```sh
admin@rbbs:~$ sudo nano /etc/fstab
```

```
lnd@raspibolt:/data/lnd /data/lnd fuse.sshfs noauto,x-systemd.automount,_netdev,reconnect,identityfile=/home/lnd/.ssh/id_rsa,allow_other,default_permissions 0 0

```

Das Dateisystem auf ```@raspibolt:/data/lnd```wird beim Systemstart mit dem Benutzer ```lnd@raspibolt``` und dem Key unter ```/home/lnd/.ssh/id_rsa``` verbunden. Mit der der Option ```allow_other``` wird das verbundene Verzeichnis auch für andere Benutzer zugänglich. Dabei werden mit der Option ```default_permissions``` die Benutzer- und Gruppenrechte vom Original übernommen.

Mit der Option ```_netdev``` wird systemxd angewiesen mit diesem Mount die Verfügbrkeit des Netzwerks abzuwarten.

lnd@raspibolt:/data/lnd /data/lnd fuse.sshfs uid=lnd,gid=lnd,x-systemd.automount,_netdev,reconnect,identityfile=/home/lnd/.ssh/id_rsa,allow_other,default_permissions 0 0

Mount:

systemd-1 on /data/lnd type autofs (rw,relatime,fd=54,pgrp=1,timeout=0,minproto=5,maxproto=5,direct,pipe_ino=11562)



Damit die Option ```allow_other``` verwendet werden darf, ist in der Datei ```fuse.conf```die Option ```user_allow_other```zu aktivieren:

```sh
admin@rbbs:~$ sudo nano /etc/fuse.conf
```

Kommentar bei ```user_allow_other``` entfernen.



Das File-System neu laden:

```sh
admin@rbbs:~$ systemctl daemon-reload
```



Aus dem fstab - Eintrag werden zwei systemd - Mount - Unit generiert:

/run/systemd/generator/data-lnd.automount

/run/systemd/generator/data-lnd.moun

Mit diesen Units wird der Mount mmit dem User root gemacht. Damit dies gelinget nuss auch de User Root ein Key besitzen:

admin@rbbs:~$ sudo su -

root@rbbs:~# ssh-keygen -t rsa -b 4096

Public Key in authen..File von lnd@raspibolt aufnehmen



Nun unktioniert es nach:

admin@rbbs:~$ sudo systemctl start data-lnd.mount



Weiter verfolgen, Sonst anstelle von Mount als service starten

https://wiki.ubuntuusers.de/systemd/Units/

https://wiki.ubuntuusers.de/systemd/Mount_Units/

https://wiki.ubuntuusers.de/systemd/Service_Units/





Script-Datei für Mount von ```/data/lnd/```erstellen

```sh
sudo nano /usr/local/bin/mount_data_lnd
```

Text einfügen, speichern und verlassen:

```
sshfs lnd@raspibolt:/data/lnd /data/lnd -o reconnect,identityfile=/home/lnd/.ssh/id_rsa,allow_other,default_permissions
```

Datei ausführbar machen:

```sh
sudo chmod +x /usr/local/bin/mount_data_lnd
```



As user “admin”, create the service file

```
$ sudo nano /etc/systemd/system/mount_data_lnd.service
```

Paste the following configuration. Save and exit

```
# rbbs: systemd unit for mount the LND-Data-Directory from RaspiBolt
# /etc/systemd/system/mount_data_lnd.service

[Unit]
Description=Mount LND Datadirectory from RaspiBolt
After=network.target 
After=network-online.target

[Service]
WorkingDirectory=/home/btcrpcexplorer/btc-rpc-explorer
ExecStart=/usr/local/bin/mount_data_lnd start
User=lnd

Restart=always
RestartSec=30

[Install]
WantedBy=multi-user.target
```

Enable the service, start it and check the log output

```
$ sudo systemctl enable btcrpcexplorer.service
$ sudo systemctl start btcrpcexplorer.service
$ sudo journalctl -f -u btcrpcexplorer
```



Der folgende Test war erfolgreich:

admin@rb-bonus:~$ sudo mkdir /data/lndadmin
admin@rb-bonus:~$ sudo chown -R admin:admin /data/lndadmin
admin@rb-bonus:~$ sshfs -o debug admin@raspibolt:/data/lnd /data/lndadmin
SSHFS version 3.7.1
executing <ssh> <-x> <-a> <-oClearAllForwardings=yes> <-2> <admin@raspibolt> <-s> <sftp>
Server version: 3
Extension: posix-rename@openssh.com <1>
Extension: statvfs@openssh.com <2>
Extension: fstatvfs@openssh.com <2>
Extension: hardlink@openssh.com <1>
Extension: fsync@openssh.com <1>
Extension: lsetstat@openssh.com <1>









