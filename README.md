# vpn-access-for-bonus-guide
POC: VPN for bonus guide tools access to the raspibolt

Ausgangslage:

Im Bonusguide des Projekts RASPIBOLT werden in wachsender Anzahl neue Services und Tolls eingeführt. Diese werden auf dem Raspibolt insalliert. Dies führt zu folgenden Problemen:
- steigende Auslastung der Resourcen
- die kritischen Node-Basis-Dienste (bitcoind electrs und lnd) werden durch Installation und Updates der zusätzlichen Services und Tools in der Verfügbarkeit gefärdet.


Ziel des POC

Es soll ein Vorschlag erarbeitet werden wie die Servicese und Tools des Bonus-guides und auch der Basis-Dienst RTL und Blickchin Explorer auf einer dedizierten HW betriben werden kann. Der Zugriff soll einfach aber sicher erfolgen.

