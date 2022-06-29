# Testnet

Mit einer Testnet-Installation können neue Applikationen getestet werden. Motivation für dies Installation war der Wunsch die Monitot App lndmon zu installieren.

Die Installation erfolgt gemäss 

[Raspibolt]: https://raspibolt.org

unter Berücksichtigungen der Änderungen im Bonusguide:  https://raspibolt.org/guide/bonus/bitcoin/testnet.html

1. Tor-Installation
   1. Achtung: aktuell muss Tor nach der Installation noch updatet werden:
      1. sudo apt dist-upgrade
      2. tor --Version
      3. Tor Version 0.4.7.7. (Bei mir wurde immer noch die Version 0.4.5.10. angezeigt. Ev. weil es nicht Raspi sonder Debian ist.)
2. bitcoind-Installation
   1. https://raspibolt.org/guide/bitcoin/bitcoin-client.html
3. Electrum-Server
   1. NGINX und FW
   2. Electrs
      1. sudo apt-get install cargo (Cargo wird zur Compilation  benötigt)
      2. sudo apt-get install libclang-dev (wird von Cargo zur Compilation benötigt)



