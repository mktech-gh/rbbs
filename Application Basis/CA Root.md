Installation CA - Root

Mit der Installation einer CA auf **rbbs.local** werden vertrauenswürdige Serverzertifikate signiert. Dies ermöglicht einen Zugriff auf **RaspiBolt** ohne Fehlermeldung.

Inspiration: https://checkmk.com/de/linux-wissen/ca-zertifikat-erstellen

Voraussetzung:

- sshd ist installiert und läuft

---

Mit **rbbs** mit Benutzer admin verbinden, Benutzer wechseln auf root und Ordner einrichten

```ssh
admin@rbbs:~/ca# sudo su -
root@rbbs:~/ca# mkdir /root/ca
root@rbbs:~/ca# mkdir /root/ca/certs
root@rbbs:~/ca# cd /root/ca
```

Default-Wert der CA modifizieren

```ssh
root@rbbs:~/ca# nano /etc/ssl/openssl.cnf
```

CA Zertifikat generieren

```ssh
root@rbbs:~/ca# openssl req -new -x509 -newkey rsa:4096 -keyout ca-mktech-key.pem -out ca-mktech-cert.pem -days 3650
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

Neue Zertifikate installieren um die Verbindung von Windows-Firefox nach raspibolt.local ohne Fehlermeldung zu verwenden:

1. Root-Zertifikat ```ca-mktech-cert.pem``` auf Windows-Client importieren (Vertrauenswürdige Stammzertifizierungsstellen)

2. Firefox Browser für den Windows-Zertifikatsspeicher-Zugriff konfigurieren:

   1. URL: about:config

   2. ```security.enterprise_roots.enabled``` auf **True** stellen.

3. ```raspibolt-local-cert.pem```  nach ```raspibolt:/etc/ssl/certs``` kopieren

4. ```raspibolt-local-key.pem``` nach ```raspibolt:/etc/ssl/private``` kopieren


NGINX-Konfiguration von ``` ss ``` auf ```raspibolt-local-key.pem``` ändern:

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
admin@raspibolt:~ $ sudo systemctl sshd
```


