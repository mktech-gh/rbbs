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

The User connect via ``rbbs:NGINX`` to ``RaspiBolt:BTC RPC Explorer SSL`` 

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
sudo su -
mkdir /root/ca
cd /root/ca
openssl req -new -x509 -newkey rsa:4096 -keyout ca.mktech-key.pem -out ca-mktech-cert.pem -days 3650
```

PENDENZ:









sudo ufw allow log-all in on wlan0 from 192.168.1.234

net stop npcap
