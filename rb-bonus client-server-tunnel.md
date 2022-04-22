---
layout: default
title: Bitcoin client
nav_order: 10
parent: Bitcoin
---
<!-- markdownlint-disable MD014 MD022 MD025 MD033 MD040 -->
# Bitcoin client
{: .no_toc }



## Installation und Start verschlüsselter Tunnel

Pendenzen allgemein
###################
1. Access auf rb-bonus von PC-vomfass ebenfalls mit SSH-Keys einrichten
2. WLAN-Aktivierung automatisieren, damit nicht bei jedem Neustart wieder auf der Konsole eingeloggt werden muss und das WLAN verbunden werden muss.

Variante sshuttle
#################

https://sshuttle.readthedocs.io/en/stable/installation.html

$ sudo apt install sshuttle

Neuer User admin anlegen 
https://raspibolt.org/guide/raspberry-pi/system-configuration.html#add-the-admin-user-and-log-in-with-it

Login auf RaspiBolt mit SSH-Keys konfigurieren
https://raspibolt.org/guide/raspberry-pi/security.html#login-with-ssh-keys

Test Login mit SSH-Key
$ ssh admin@raspibolt

Beim ersten Zugriff wird der Fingerprint zur Überprüfung angezeigt.

sshuttle verbinden
$ sshuttle -r admin@raspibolt 0.0.0.0/0

Mit Ctrl C abbrechen

Pendenzen:
1. ev. noch sshuttle --sudoers ausführen. siehe: https://sshuttle.readthedocs.io/en/stable/manpage.html#cmdoption-sshuttle-sudoers
2. mit nfs ein Laufwerk zu mounten


Variante sshfs
##############

https://linuxize.com/post/how-to-use-sshfs-to-mount-remote-directories-over-ssh/

Installing SSHFS

SSHFS is available from the default Debian repositories. Update the packages index and install the sshfs client by typing:

admin@rb-bonus~$ sudo apt update
admin@rb-bonus~$ sudo apt install sshfs

Analog RaspiBolt wird ein Benutzer lnd und das Datendirectory /data/lnd hinzugefügt
Auf das lokale Directory /data/lnd wird das RaspiBolt-Directory /data/lnd gemountet

admin@rb-bonus~$ sudo adduser --disabled-password --gecos "" lnd

Pendenz: Falls Tor installiert wird, muss lnd eventuell noch zur Gruppe debian-tor hinzugefügt werden
Pendenz: Danicht mit dem Benutzer lnd auf rb-bonus gearbeitet wird, ist diese Gruppenzuordnung voraussichtlich nicht nötig
Pendenz: admin@rb-bonus~$  sudo usermod -a -G debian-tor lnd


Add the user “admin” to the group “lnd”

admin@rb-bonus~$ sudo adduser admin lnd

create the LND data directory

admin@rb-bonus~$ sudo mkdir /data
admin@rb-bonus~$ sudo mkdir /data/lnd
admin@rb-bonus~$ sudo chown -R lnd:lnd /data/lnd

Use the sshfs command to mount the remote directory (-o debug nur zum testen):

admin@rb-bonus~$ sudo su - lnd
lnd@rb-bonus~$ sshfs -o debug lnd@raspibolt:/data/lnd /data/lnd

Pendenz:
user lnd@rb-bonus muss noch public key mit user lnd@raspibolt austauschen
Aktuell gibt es im debugmodus noch die folgende Fehlermeldung
lnd@rb-bonus:~$ sshfs -o debug lnd@raspibolt: /data/lnd
SSHFS version 3.7.1
executing <ssh> <-x> <-a> <-oClearAllForwardings=yes> <-2> <lnd@raspibolt> <-s> <sftp>
lnd@raspibolt: Permission denied (publickey).
read: Connection reset by peer

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









