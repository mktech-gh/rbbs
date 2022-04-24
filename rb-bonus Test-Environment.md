# rb-bonus

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
- Keine zusätzliche Software wird installiert. Auch nicht die vorgeschlagenen Pakete: Debian Desktop enviroment und GNOME
- Laptop beim zuklappen des Deckels nicht ausschalten

```sh
sudo nano /etc/systemd/logind.conf
```

Paste the following lines. Save and exit.

```ini
HandleLidSwitch=ignore
```
  ```sh
$ sudo service systemd-logind restart
  ```

## config wlan with wpa_supplicant 
   ```sh
   $ sudo nano /etc/wpa_supplicant/wpa_supplicant.conf
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
  $ wpa_passphrase "WLAN-NAME" "WLAN-PASSWORT" >> /etc/wpa_supplicant/wpa_supplicant.conf
  $ exit
   ```
Überprüfen von wpa_supplicant.conf:
  ```sh
  $ sudo cat /etc/wpa_supplicant/wpa_supplicant.conf
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
  WLAN-Interface aktivieren, am Acccesspoint anmelden und IP-Adresse beziehen
  ```sh
  $ sudo /sbin/ifup -a -v
  $ sudo wpa_supplicant -i wls1 -c /etc/wpa_supplicant/wpa_supplicant.conf &
  $ sudo dhclient
  ```
### WLAN beim Booten automatisch aktivieren
  Backup von `interfaces` erstellen und `interfaces` anpassen
  ```sh
  $ sudo cp /etc/network/interfaces /etc/network/interfaces.back
  $ sudo nano /etc/network/interfaces
  ```
  Paste the following lines. Save and exit.
  ```ini
  allow-hotplug wls1
  iface wls1 inet dhcp
  pre-up wpa_supplicant -i wls1 -c /etc/wpa_supplicant/wpa_supplicant.conf
  ```

  - ssh-server aktivieren:
  ```sh
  $ sudo apt install openssh-server
  $ sudo systemctl status ssh
  $ sudo service ssh stop
  $ sudo service ssh start
  ```
  Nun ist das System bereit für den POC

