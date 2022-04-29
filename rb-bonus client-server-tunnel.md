# Installation und Aktivierung verschlüsselter Tunnel

Pendenzen allgemein

1. 

## Generell für alle Varianten

Neuer User `admin` anlegen 
https://raspibolt.org/guide/raspberry-pi/system-configuration.html#add-the-admin-user-and-log-in-with-it

Verzeichnis `/data`erstellen

```sh
admin@rb-bonus~$ sudo mkdir /data
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
admin@rb-bonus~$ sudo apt update
admin@rb-bonus~$ sudo apt install sshfs
```

Analog RaspiBolt wird ein Benutzer lnd und das Datendirectory /data/lnd hinzugefügt

- Auf dieses lokale Directory /data/lnd wird später das RaspiBolt-Directory /data/lnd gemountet

```sh 
admin@rb-bonus~$ sudo adduser --disabled-password --gecos "" lnd
admin@rb-bonus~$ sudo mkdir /data/lnd
admin@rb-bonus~$ sudo chown -R lnd:lnd /data/lnd
```

- Pendenz: Falls Tor installiert wird, muss lnd eventuell noch zur Gruppe debian-tor hinzugefügt werden
  Pendenz: Danicht mit dem Benutzer lnd auf rb-bonus gearbeitet wird, ist diese Gruppenzuordnung voraussichtlich nicht nötig
  Pendenz: admin@rb-bonus~$  sudo usermod -a -G debian-tor lnd

- Add the user “admin” to the group “lnd”

```sh
admin@rb-bonus~$ sudo adduser admin lnd
```



### lnd Key exchange

Mit User ``lnd`` Key generieren, damit sich dieser Benutzer beim Raspibolt für den mount des Laufwerks anmelden kann.

-  Schlüssel generieren (`/home/lnd/.ssh/id_rsa`) und auf `admin@raspibolt/home/asdmin/.ssh/authorized_keys` hinzufügen

```sh
admin@rb-bonus~$ sudo su - lnd
lnd@rbbs~$ ssh-keygen -t rsa -b 4096
lnd@rbbs:~$ cat .ssh/id_rsa.pub
```

- angezeigter Public-Key von `` lnd@rbbs`` kopieren.
- Auf admin@raspibolt den Benutzer auf lnd wechseln und die Datei ``authorized_keys``erstellen und den pulic-key von lnd@rbbs einfügen

```sh
admin@raspibolt:~$ sudo su - lnd
lnd@raspibolt:~ $ nano authorized_keys
```

### Manual Mount the remote lnd Directory

Das  remote directory mit dem sshfs Befehl verbinden (-o debug zum testen):

```sh
lnd@rb-bonus~$ sshfs -o debug lnd@raspibolt:/data/lnd /data/lnd
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

Dazu wird die Datei /etc/fstab ergänzt:

```sh
admin@rbbs:~$ sudo nano /etc/fstab
```

```
lnd@raspibolt:/data/lnd /data/lnd fuse.sshfs noauto,x-systemd.automount,_netdev,reconnect,identityfile=/home/lnd/.ssh/id_rsa,allow_other,default_permissions 0 0
```

Das File-System neu laden:

```sh
admin@rbbs:~$ systemctl daemon-reload
```



```sh
lnd@raspibolt:/data/lnd/ /data/lnd fuse.sshfs noauto,x-systemd.automount,_netdev,reconnect,identityfile=/home/sammy/.ssh/id_rsa,allow_other,default_permissions 0 0
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









