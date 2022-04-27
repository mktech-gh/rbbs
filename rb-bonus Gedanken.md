#Connection Varianten

Anforderungen an die Verbindung

Werden die Services und Tools auf entfernter HW installiert, muss die Verbindung zwischen dieser HW und dem Raspibolt folgende MErkmale aufweisen:
- verschlüsselt
- automatischer Aufbau nach dem Aufstarten eines oder beider Systeme.


Varianten

SSH-Portforwarding

- openssh's port forwarding
- sshuttle https://github.com/apenwarr/sshuttle


File Share over Tunnel
https://gist.github.com/proudlygeek/5721498
https://journal.mach5.web.id/2019/09/nfs-via-ssh-tunnel

File Sahre direct (Müsste mit sshuttle funktionieren)
https://linuxize.com/post/how-to-mount-an-nfs-share-in-linux
https://linuxize.com/post/how-to-install-and-configure-an-nfs-server-on-ubuntu-18-04

Fileshare über sshfs
https://linuxize.com/post/how-to-use-sshfs-to-mount-remote-directories-over-ssh



