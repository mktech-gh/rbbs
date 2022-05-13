# Installation der Bonus-Applikationen
```
Dieses Tutorial beschreibt die Installation auf dem dedizierten rbbs-Server. Dabei wird auf den RaspiBolt Guide referenziert und in diesem Dokument nur die Besonderheit der dedizierten Installation beschrieben.
```

---

## Enabling the Uncomplicated Firewall

Installation according to:https://raspibolt.org/guide/raspberry-pi/security.html#enabling-the-uncomplicated-firewall

### Exceptions to the guide

- No exceptions

---

## NGINX

Installation according to: https://raspibolt.org/guide/raspberry-pi/security.html#prepare-nginx-reverse-proxy

### Exceptions to the guide

- No exceptions

___

## Blockchain explorer

Installation according to: https://raspibolt.org/guide/bitcoin/blockchain-explorer.html#blockchain-explorer

The User connect via ``rbbs:NGINX`` to ``RaspiBolt:BTC RPC Explorer SSL `` via ```RASPIBolt:NGINX```

Die Verbindung muss via ```RASPIBolt:NGINX``` erfolgen:

- ``RaspiBolt:BTC RPC Explorer SSL `` hört auf 127.0.0.1 und somit nur auf lokale  Anfragen
- SSL Verschlüsselung. 

### Exceptions to the guide

- Skip Installation from Node.js

- NGINX

  - change in NGINX-Configuration the Adress from 127.0.0.1 to the RaspiBolt-Adress (192.168.1.233)

    ```shell
    admin@rbbs:~$ sudo nano /etc/nginx/streams-enabled/btcrpcexplorer-reverse-proxy.conf
    ```

    ```
    upstream btcrpcexplorer {
      server 192.168.1.234:4000;
    }
    server {
      listen 4000 ssl;
      proxy_pass btcrpcexplorer;
    }
    ```

    

on raspibolt add Firewall-Rule for Connection from rbbs:nginx to raspibolt:BTC RPC Explorer

```sh
admin@raspibolts:~$ sudo ufw allow in on wlan0 from 192.168.1.234  to 192.168.1.233 port 3002 comment "allow BTC RPC Explorer SSL from rbbs"
```

Test auf rbbs:

https://100.107.147.15:4000/

-> Fehler: Gesicherte Verbindung fehlgeschlagen

Test auf raspibolt:

https://100.86.152.52:4000/

-> funktioniert



Weitere Abklärungen mit tcpdump

```sh
admin@rbbs:~$ sudo apt install tcpdump
```

Weiteres Vorgehen:

Grundsätzlich wird folgendes Verfahren angestrebt/weiterverfolgt:

Browser -> rbbs:NGINX --> raspibolt:NGINX --> Applikation

Problem-Annahme:

- rbbs:NGINX akzeptiert Zertifikat von  raspibolt:NGINX nicht

https://docs.nginx.com/nginx/admin-guide/security-controls/terminating-ssl-http/

https://docs.nginx.com/nginx/admin-guide/security-controls/securing-http-traffic-upstream/

CA erstellen um eigene Zertifikate zu erstellen:

https://checkmk.com/de/linux-wissen/ca-zertifikat-erstellen

```sh
root@rbbs:~/ca# sudo su -
root@rbbs:~/ca# mkdir /root/ca
root@rbbs:~/ca# mkdir /root/ca/certs
root@rbbs:~/ca# cd /root/ca
# Default-Wert der CA modifizieren
root@rbbs:~/ca# nano /etc/ssl/openssl.cnf
#CA Zeritifikat generieren:
root@rbbs:~/ca# openssl req -new -x509 -newkey rsa:4096 -keyout ca-mktech-key.pem -out ca-mktech-cert.pem -days 3650
# Private-Key für Server generieren mit Passphase
root@rbbs:~/ca# openssl genrsa -aes128 2048 > raspibolt-local-key.pem
# Passphrase entfernen
root@rbbs:~/ca# openssl rsa -in raspibolt-local-key.pem -out raspibolt-local-key.pem
# Cert-Request für raspibold.local erstellen
root@rbbs:~/ca# openssl req -new -key raspibolt-local-key.pem -out raspibolt-local-cert-req.pem -nodes
#Files für Zähler und Protokoll anlegen
root@rbbs:~/ca# touch index.txt
root@rbbs:~/ca# echo 01 > serial
#Zertifikat ausstellen für raspibolt.local
root@rbbs:~/ca# openssl ca -in raspibolt-local-cert-req.pem -notext -out raspibolt-local-cert.pem
            Using configuration from /usr/lib/ssl/openssl.cnf
            Enter pass phrase for ./ca-mktech-key.pem:
            Check that the request matches the signature
            Signature ok
            Certificate Details:
                    Serial Number: 1 (0x1)
                    Validity
                        Not Before: May 13 09:40:58 2022 GMT
                        Not After : May 10 09:40:58 2032 GMT
                    Subject:
                        countryName               = CH
                        organizationName          = MKTECH
                        organizationalUnitName    = MKTECH
                        commonName                = raspibolt.local
                        emailAddress              = mktech-info@riseup.net
                    X509v3 extensions:
                        X509v3 Basic Constraints:
                            CA:FALSE
                        Netscape Cert Type:
                            SSL Client, S/MIME
                        X509v3 Key Usage:
                            Digital Signature, Non Repudiation, Key Encipherment
                        Netscape Comment:
                            OpenSSL Generated Certificate
                        X509v3 Subject Key Identifier:
                            7B:73:B7:A4:D4:77:30:12:8A:22:75:09:3E:B3:6D:C8:53:2A:0E:7E
                        X509v3 Authority Key Identifier:
                            keyid:C0:A7:AF:8E:84:DB:FC:60:19:F5:41:94:D7:0D:48:81:20:9B:73:59

            Certificate is to be certified until May 10 09:40:58 2032 GMT (3650 days)
            Sign the certificate? [y/n]:y


            1 out of 1 certificate requests certified, commit? [y/n]y
            Write out database with 1 new entries
            Data Base Updated


```

Neue Zertifikate installieren um die Verbindung Windows-Firefox nach raspibolt.local zu benutzen:

1. Root-Zertifikat ```ca-mktech-cert.pem``` auf Windows-Client importieren (Vertrauenswürdige Stammzertifizierungsstellen)

2. Firefox Browser für den Windows-Zertifikatsspeicher-Zugriff konfigurieren:

   1. About:config

   2. ```security.enterprise_roots.enabled``` auf **True** stellen.

3. ```raspibolt-local-cert.pem```  nach ```raspibolt:/etc/ssl/certs``` kopieren

4. ```raspibolt-local-key.pem``` nach ```raspibolt:/etc/ssl/private``` kopieren

5. NGINX-Konfiguration von ``` ss ``` auf ```raspibolt-local-key.pem``` umkonfigurieren:

   ```sh
   sudo nano /etc/nginx/nginx.conf
   ```

> \#  ssl_certificate /etc/ssl/certs/nginx-selfsigned.crt;
>
> \# ssl_certificate_key /etc/ssl/private/nginx-selfsigned.key;
>
>  ssl_certificate /etc/ssl/certs/raspibolt-local-cert.pem;
>
>  ssl_certificate_key /etc/ssl/private/raspibolt-local-key.pem;'

```
admin@raspibolt:~ $ sudo nginx -t
admin@raspibolt:~ $ sudo systemctl restart nginx.service

```





PENDENZ:





sudo ufw allow log-all in on wlan0 from 192.168.1.234

net stop npcap
