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



CA erstellen um eigene Zertifikate zu erstellen:

https://checkmk.com/de/linux-wissen/ca-zertifikat-erstellen

```sh
root@rbbs:~/ca# sudo su -
root@rbbs:~/ca# mkdir /root/ca
root@rbbs:~/ca# cd /root/ca
#CA Zeritifikat generieren:
root@rbbs:~/ca# openssl req -new -x509 -newkey rsa:4096 -keyout ca-mktech-key.pem -out ca-mktech-cert.pem -days 3650
# Private-Key für Server generieren mit Passphase
root@rbbs:~/ca# openssl genrsa -aes128 2048 > serverkey-raspbolt.pem
# Passphrase entfernen
root@rbbs:~/ca# openssl rsa -in serverkey-raspbolt.pem -out serverkey-raspbolt.pem
# Cert-Request für raspibold.local erstellen
root@rbbs:~/ca# openssl req -new -key serverkey-raspbolt.pem -out req-raspibolt-cert.pem -nodes
# Defailt-Werteder CA modifizieren
root@rbbs:~/ca# nano /etc/ssl/openssl.cnf
        dir             = .                     # Where everything is kept
        certs           = $dir/certs            # Where the issued certs are kept
        crl_dir         = $dir/crl              # Where the issued crl are kept
        database        = $dir/index.txt        # database index file.
        #unique_subject = no                    # Set to 'no' to allow creation of
                                                # several certs with same subject.
        new_certs_dir   = $dir                  # default place for new certs.

        certificate     = $dir/ca-mktech-cert.pem # The CA certificate
        serial          = $dir/serial           # The current serial number
        crlnumber       = $dir/crlnumber        # the current crl number
                                                # must be commented out to leave a V1 CRL
        crl             = $dir/crl.pem          # The current CRL
        private_key     = $dir/ca-mktech-key.pem # The private key
        RANDFILE        = $dir/.rand            # private random number file insert from mk

        x509_extensions = usr_cert              # The extensions to add to the cert
#Files für Zähler und Protokoll anlegen
root@rbbs:~/ca# touch index.txt
root@rbbs:~/ca# echo 01 > serial
#Zertifikat ausstellen für raspibolt.local
root@rbbs:~/ca# openssl ca -in req-raspibolt-cert.pem -notext -out raspibolt-local-cert.pem
        Using configuration from /usr/lib/ssl/openssl.cnf
        Enter pass phrase for ./ca-mktech-key.pem:
        Check that the request matches the signature
        Signature ok
        Certificate Details:
                Serial Number: 1 (0x1)
                Validity
                    Not Before: May 12 20:04:48 2022 GMT
                    Not After : May  9 20:04:48 2032 GMT
                Subject:
                    countryName               = CH
                    organizationName          = MKTECH
                    commonName                = raspibolt.local
                X509v3 extensions:
                    X509v3 Basic Constraints:
                        CA:FALSE
                    Netscape Comment:
                        OpenSSL Generated Certificate
                    X509v3 Subject Key Identifier:
                        86:02:4B:B9:E7:C6:51:F4:8B:45:38:04:9F:7B:CA:44:CF:54:22:1C
                    X509v3 Authority Key Identifier:
                        keyid:52:B5:85:98:27:D3:6F:05:66:2C:CF:B9:46:76:10:36:A3:C6:4D:68

        Certificate is to be certified until May  9 20:04:48 2032 GMT (3650 days)
        Sign the certificate? [y/n]:y


        1 out of 1 certificate requests certified, commit? [y/n]y
        Write out database with 1 new entries
        Data Base Updated


```

PENDENZ:

Zertifikatserstellung wiederholen: Zuerst noch root@rbbs:~/ca# nano /etc/ssl/openssl.cnf mit meinen Deafult-Werten (Möglichst genau) optimieren





sudo ufw allow log-all in on wlan0 from 192.168.1.234

net stop npcap
