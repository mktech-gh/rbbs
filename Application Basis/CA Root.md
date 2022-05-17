Installation CA - Root

Mit der Installation einer CA auf **rbbs.local** werden vertrauenswürdige Serverzertifikate signiert. Dies ermöglicht einen Zugriff auf **RaspiBolt** ohne Fehlermeldung.

Inspiration: https://checkmk.com/de/linux-wissen/ca-zertifikat-erstellen

Voraussetzung:

- sshd ist installiert und läuft

---

Mit **rbbs** mit Benutzer admin verbinden, 

Defaultwerte der CA festlegen:

```sh
admin@rbbs:~$ nano /etc/ssl/openssl.cnf
```

Benutzer ca erstellen, Datei ```openssl.cnf```für die Gruppe ca lesbar machen, zu Benutzer ca wechseln und Ordner einrichten

```ssh
admin@rbbs:~$ sudo adduser --disabled-password --gecos "" ca
admin@rbbs:$ sudo chown :ca /etc/ssl/openssl.cnf
admin@rbbs:~$ sudo su - ca
root@rbbs:~# mkdir /home/ca/ca
root@rbbs:~# mkdir /home/ca/ca/certs
root@rbbs:~# cd /home/ca/ca
```

CA Zertifikat generieren

```ssh
root@rbbs:~/ca# openssl req -new -x509 -newkey rsa:4096 -keyout ca-mktech-key.pem -out ca-mktech-cert.pem -days 3650
```

```
Generating a RSA private key
................++++
...................................................................................................................................................................................................................................++++
writing new private key to 'ca-mktech-key.pem'
Enter PEM pass phrase:
Verifying - Enter PEM pass phrase:
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [CH]:
State or Province Name (full name) []:.
Locality Name (eg, city) []:.
Organization Name (eg, company) [MKTECH]:
Organizational Unit Name (eg, section) [MKTECH]:
Common Name (e.g. server FQDN or YOUR name) []:ca-mktech
Email Address [mktech-info@riseup.net]:
```

Private-Key mit Passphase für Serverzertifikat generieren. Anschliessend Passphrase entfernen

```ssh
root@rbbs:~/ca# openssl genrsa -aes128 2048 > raspibolt-local-key.pem
root@rbbs:~/ca# openssl rsa -in raspibolt-local-key.pem -out raspibolt-local-key.pem
```

Cert-Request für raspibold.local erstellen. Der common-name muss zwingend die Server-URL sein.

```ssh
root@rbbs:~/ca# openssl req -new -key raspibolt-local-key.pem -out raspibolt-local-cert-req.pem -nodes
```

Files für Zähler und Protokoll (CA-Datenbank) anlegen

```ssh
root@rbbs:~/ca# touch index.txt
root@rbbs:~/ca# echo 01 > serial
```

Zertifikat ausstellen für raspibolt.local

```ssh
root@rbbs:~/ca# openssl ca -in raspibolt-local-cert-req.pem -notext -out raspibolt-local-cert.pem
root@rbbs:~/ca# exit
```

### Clients vorbereiten 

Neues CA-Root-Zertifikate auf den Clients installieren um die Server raspibolt.local und rbbs.local als Vertrauenswürdig zu erkennen.

#### Windows mit Firefox als Client verwenden:

1. Root-Zertifikat ```ca-mktech-cert.pem``` auf Windows-Client kopieren und in den Zertifikatsspeicher importieren (Vertrauenswürdige Stammzertifizierungsstellen)
   1. run --> mmc c:\windows\system32\certlm.msc

   2. Zertifikate - Local Computer\Vertrauenswürdige Stammzertifikate\Zertifikate

   3. rechte Maustaste auf Zertifikate  -> Alle Aufgaben -> Importieren...

   4. ```ca-mktech-cert.pem```auswählen und importieren

2. Firefox Browser für den Windows-Zertifikatsspeicher-Zugriff konfigurieren:

   1. URL: about:config

   2. ```security.enterprise_roots.enabled``` auf **True** stellen.

   3. Firefox neu starten

#### Raspberry Pi als Client benutzen

Root-Zertifikat ```ca-mktech-cert.pem``` in Zertifikatsspeicher kopieren und umbenennen. es muss die Endung .crt haben, damit es vom nachfolgenden Programm erkannt wird.

```sh
admin@rbbs:~$ sudo mkdir /usr/share/ca-certificates/ca-mktech
admin@rbbs:~$ sudo cp /home/ca/ca/ca-mktech-cert.pem /usr/share/ca-certificates/ca-mktech/ca-mktech-cert.crt
```

Root-Zertifikat ```ca-mktech-cert.pem``` kompilieren

~~~sh
~~~





#### Zertifikat und Key für NGINX bereitstellen

```raspibolt-local-cert.pem```  nach ```raspibolt:/etc/ssl/certs``` kopieren

```sh
ca@rbbs:~/ca$ cat raspibolt-local-cert.pem
```

Zertifikat in Zwischenablage kopieren und auf ```raspibolt``` einfügen

```sh
admin@raspibolt:~$ sudo nano /etc/ssl/certs/raspibolt-local-cert.pem
```



```raspibolt-local-key.pem``` nach ```raspibolt:/etc/ssl/private``` kopieren

```sh
ca@rbbs:~/ca$ cat raspibolt-local-key.pem
```

Key in Zwischenablage kopieren und auf ```raspibolt``` einfügen

```sh
admin@raspibolt:~$ sudo nano /etc/ssl/private/raspibolt-local-key.pem
```



#### NGINX anpassen

NGINX-Konfiguration von ``` nginx-selfsigned ``` auf ```raspibolt-local``` ändern:

```sh
admin@raspibolt:~ $ sudo nano /etc/nginx/nginx.conf
```

```
# ssl_certificate /etc/ssl/certs/nginx-selfsigned.crt;
# ssl_certificate_key /etc/ssl/private/nginx-selfsigned.key;
ssl_certificate /etc/ssl/certs/raspibolt-local-cert.pem;
ssl_certificate_key /etc/ssl/private/raspibolt-local-key.pem;
```

Neustart NGINX und sshd

```
admin@raspibolt:~ $ sudo nginx -t
admin@raspibolt:~ $ sudo systemctl restart nginx.service
admin@raspibolt:~ $ sudo systemctl restart sshd
```

