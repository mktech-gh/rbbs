# rb-bonus Test Environment

Aktuell sind raspbery pi nicht lieferbar. Aus diesem Grund setze ich auf einem alten Laptop ein Debian-System auf.
Ziel ist es, ein Raspberry Pi analog dem System für den RaspiBolt aufzusetzen.

HW: ASUS Laptop P52F (very old ;-)
OS:Debian
- Installation Bootloader auf USB mit https://unetbootin.github.io/

##Boot on USB. Anmerkungen zur Installation:

- Installation startet unmittelbar
- Ich gebe kein Passwort für Root. Dadurch wird root disabled und der initial definierte User bekommt die Berechtigung sudo zu benutzen
- User: mktech-admin UID: mktech-admin PW: Password[A]
- Disk Partitionierung: Ganzer Disk mit LVM (ohne verschlüsselung)
- Eine Partition (Keine separaten für /home und /tmp) 
- 100GB der verfügbaren 500GB werden partitioniert: Eine Erweiterung der Partition ist jederzeit mit dem LVM-Tool möglich
- Keine zusätzliche Software wird installiert. Auch keine vorgeschlagene Pakete: Debian Desktop environment und GNOME
- Laptop beim zuklappen des Deckels nicht ausschalten

```sh
$ sudo nano /etc/systemd/logind.conf
```

Paste the following lines. Save, exit and restart.

```ini
HandleLidSwitch=ignore
```
  ```sh
$ sudo service systemd-logind restart
  ```

## config wlan with wpa_supplicant 

Datei wpa_supplicant-wls1.conf erstellen. Wichtig den Interfacenamen mit Bindestrich zu verwenden. ```-wls1```

   ```sh
   $ sudo nano /etc/wpa_supplicant/wpa_supplicant-wls1.conf
   ```
   Paste the following lines. Save and exit.
   ```ini
   country=CH
   ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
   ap_scan=1
   update_config=1
   ```

Damit das Passwort nicht im Klartext einsehbar ist werden SSID und Passwort nicht direkt in die Datei `wpa_supplicant.conf` geschrieben. Mit dem command `wpa_passphrase`  wird SSID und der Hash des Passworts in die Datei `wpa_supplicant.conf` geschrieben. Dieser Befehl muss als `root` ausgeführt werden!
   ```sh
  $ sudo -i
  $ wpa_passphrase "WLAN-NAME" "WLAN-PASSWORT" >> /etc/wpa_supplicant/wpa_supplicant-wls1.conf
  $ exit
   ```
Überprüfen von wpa_supplicant-wls1.conf:
  ```sh
  $ sudo cat /etc/wpa_supplicant/wpa_supplicant-wls1.conf
  > country=CH
  > ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
  > ap_scan=1
  > update_config=1
  
  > network={
  >        ssid="mdq-92227"
  >        psk=748d56b8cf29278050184197d9410899baa20aed0ec65f13405b1be83d1ddd70
  > }
  ```
### WLAN manuell aktivieren und IP-Adresse beziehen
  Name des WLAN-Interface in Erfahrung bringen:
  ```sh
  $ ls /sys/class/net
  > ens5f5  lo wls1
  ```
  WLAN-Interface aktivieren, am Accesspoint anmelden und IP-Adresse beziehen
  ```sh
  $ sudo /sbin/ifup -a -v
  $ sudo wpa_supplicant -i wls1 -c /etc/wpa_supplicant/wpa_supplicant-wls1.conf &
  $ sudo dhclient
  ```
### WLAN beim Booten automatisch aktivieren

Inspiriert von: https://wiki.somlabs.com/index.php/Connecting_to_WiFi_network_using_systemd_and_wpa-supplicant

​	`networking.service`  wird deaktiviert, da der aktuellere Service `systemd-network.service`verwendet wird. Sofern ein File ```/etc/network/interfaces``` vorhanden ist, dieses umbennenen, damit Interfaces nicht parallel konfiguriert werden und zu Problemen führen..

```sh
$ sudo systemctl disable networking.service
$ sudo cp /etc/network/interfaces /etc/network/interfaces.back
```

Erstellen der `systemd-network.service` network-Konfigurationsdatei. Mit ```10 erreicht man eine hohe Priorisierung für diese Konfiguration, da die möglichen Konfigurationsfiles alphabetisch sortiert werden. Das erst bei dem der Interfacename matched wird für die Konfiguration genommen.

```sh
$ sudo nano /etc/systemd/network/10-wls1.network
```
​	Paste the following lines. Save and exit.
```
[Match]
Name=wls1

[Network]
DHCP=yes
IgnoreCarrierLoss=3s
```

​	`systemd-networkd.service` aktivieren, starten und überprüfen

```sh
admin@rbbs:~$ sudo systemctl enable systemd-networkd.service
admin@rbbs:~$ sudo systemctl start systemd-networkd.service
admin@rbbs:~$ sudo systemctl status systemd-networkd.service
```

```
● systemd-networkd.service - Network Service
     Loaded: loaded (/lib/systemd/system/systemd-networkd.service; enabled; vendor preset: enabled)
     Active: active (running) since Wed 2022-05-04 07:40:03 CEST; 2h 21min ago
TriggeredBy: ● systemd-networkd.socket
       Docs: man:systemd-networkd.service(8)
   Main PID: 278 (systemd-network)
     Status: "Processing requests..."
      Tasks: 1 (limit: 4422)
     Memory: 4.0M
        CPU: 524ms
     CGroup: /system.slice/systemd-networkd.service
             └─278 /lib/systemd/systemd-networkd

May 04 07:40:03 rbbs systemd-networkd[278]: ens5f5: Link UP
May 04 07:40:04 rbbs systemd-networkd[278]: wlan0: Interface name change detected, wlan0 has been renamed to wls1.
May 04 07:40:04 rbbs systemd-networkd[278]: wls1: Link UP
May 04 07:40:07 rbbs systemd-networkd[278]: wls1: Gained carrier
May 04 07:40:07 rbbs systemd-networkd[278]: wls1: Connected WiFi access point: mdq-92227 (ec:f4:51:f1:48:8c)
May 04 07:40:08 rbbs systemd-networkd[278]: wls1: DHCPv4 address 192.168.1.234/24 via 192.168.1.1
May 04 07:40:09 rbbs systemd-networkd[278]: tailscale0: Link UP
May 04 07:40:09 rbbs systemd-networkd[278]: tailscale0: Gained carrier
May 04 07:40:09 rbbs systemd-networkd[278]: tailscale0: Gained IPv6LL
May 04 07:40:09 rbbs systemd-networkd[278]: wls1: Gained IPv6LL
```



​	`status wpa_supplicant@wls1.service` aktivieren, starten und überprüfen
Wir verwenden die "interface-spezific version"

```sh
admin@rbbs:~$ sudo systemctl enable wpa_supplicant@wls1.service
admin@rbbs:~$ sudo systemctl start wpa_supplicant@wls1.service
admin@rbbs:~$ sudo systemctl status wpa_supplicant@wls1.service
```

```
● wpa_supplicant@wls1.service - WPA supplicant daemon (interface-specific version)
     Loaded: loaded (/lib/systemd/system/wpa_supplicant@.service; enabled; vendor preset: enabled)
     Active: active (running) since Tue 2022-05-03 22:57:50 CEST; 8min ago
   Main PID: 451 (wpa_supplicant)
      Tasks: 1 (limit: 4422)
     Memory: 4.5M
        CPU: 62ms
     CGroup: /system.slice/system-wpa_supplicant.slice/wpa_supplicant@wls1.service
             └─451 /sbin/wpa_supplicant -c/etc/wpa_supplicant/wpa_supplicant-wls1.conf -Dnl80211,wext -iwls1

May 03 22:57:51 rbbs wpa_supplicant[451]: Successfully initialized wpa_supplicant
May 03 22:57:52 rbbs wpa_supplicant[451]: wls1: CTRL-EVENT-REGDOM-CHANGE init=USER type=COUNTRY alpha2=CH
May 03 22:57:52 rbbs wpa_supplicant[451]: wls1: CTRL-EVENT-REGDOM-CHANGE init=BEACON_HINT type=UNKNOWN
May 03 22:57:53 rbbs wpa_supplicant[451]: wls1: SME: Trying to authenticate with ec:f4:51:f1:48:8c (SSID='mdq-92227' freq=2412 MHz)
May 03 22:57:53 rbbs wpa_supplicant[451]: wls1: Trying to associate with ec:f4:51:f1:48:8c (SSID='mdq-92227' freq=2412 MHz)
May 03 22:57:53 rbbs wpa_supplicant[451]: wls1: Associated with ec:f4:51:f1:48:8c
May 03 22:57:53 rbbs wpa_supplicant[451]: wls1: CTRL-EVENT-SUBNET-STATUS-UPDATE status=0
May 03 22:57:53 rbbs wpa_supplicant[451]: wls1: CTRL-EVENT-REGDOM-CHANGE init=COUNTRY_IE type=COUNTRY alpha2=DE
May 03 22:57:53 rbbs wpa_supplicant[451]: wls1: WPA: Key negotiation completed with ec:f4:51:f1:48:8c [PTK=CCMP GTK=TKIP]
May 03 22:57:53 rbbs wpa_supplicant[451]: wls1: CTRL-EVENT-CONNECTED - Connection to ec:f4:51:f1:48:8c completed [id=0 id_str=]
```





## Fix systemd-networkd-wait-online.service Problem

systemd-networkd-wait-online.service ist ein systemd-serviceunit, das wartet bis alle Interfaces aktiv sind. Timeout nach 2Minuten.  Da das Ethernet-Interface nicht eingesteckt ist, gibt es beim Start einen Timeout von 2Minuten. Abhilfe:

```sh
sudo nano /lib/systemd/system/systemd-networkd-wait-online.service
```

Die folgende Zeile mit ```--any```ergänzen. Damit muss nur ein Interface aktiv sein.

```
ExecStart=/lib/systemd/systemd-networkd-wait-online --any
```



### GIT installieren

sudo apt-get install git-core



## Tailscale installieren und zu VPN hinzufügen

Voraussetzung ist ein bestehendes Tailscale-Konto

### curl installieren

User admin

```sh
$ sudo apt update && sudo apt upgrade
$ sudo apt install curl
$ curl --version
```

### Tailscale installieren

User admin

```sh
$ curl -fsSL https://tailscale.com/install.sh | sh
```

### Tailscale anmelden

```sh
$ sudo tailscale up
```

Es wird eine URL angezeigt. Diese URL aufrufen und bei Tailscale (SSO-Account) anmelden. rb-bonus wird automatisch erkannt. Fenster schliessen. Auf rb-bonus-Konsole wird angezeigt:

```
> Success.
```

### Tailscale konfigurieren

Auf https://login.tailscale.com/admin/machines anmelden und bei rb-bonus "Key expiry" disablen.



## SSH-Server

ssh-server aktivieren:

```sh
$ sudo apt install openssh-server
$ sudo systemctl status ssh
$ sudo service ssh stop
$ sudo service ssh start
```
Login mit SSH-Key

- Public-Key von Powershell und Putty in `authorized_keys` einfügen und abspeichern

```sh
$ nano .ssh/authorized_keys
```



Nun ist das System bereit für den POC

