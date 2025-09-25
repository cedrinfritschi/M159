# M159
Anleitung: Einrichtung der Rootdomäne „wondertoys.local“
Ziel

Eine Active-Directory-Domäne mit integriertem DNS auf einem Windows Server 2019 erstellen.
Die Domäne heißt wondertoys.local und läuft auf einem einzelnen Domänencontroller NYW9DC01.

Planung

Servername: NYW9DC01
(ny = Standort, w9 = Windows Server 2019, dc = Domänencontroller, 01 = Nummer)

IP-Bereich: 10.x.1.0/24, Gateway 10.x.1.1, Domänencontroller 10.x.1.21
(x = eigene Klassenposition)

DNS-Serveradresse: 10.x.1.21 (zeigt auf sich selbst)

Functional Level: Windows Server 2016

Rollen: Alle FSMO-Rollen und Global Catalog auf NYW9DC01

Vorbereitung der VM

In VMware Player eine zweite virtuelle Festplatte mit 10 GB hinzufügen:
Add → Hard Disk → SCSI → Create a new virtual disk → 10 GB → Split into multiple files.

CD-Laufwerk auf „Physical Drive“ abkoppeln, Soundkarte entfernen.

Windows Server installieren und Grundupdates durchführen.

Grundkonfiguration Windows Server

Hostname ändern auf NYW9DC01 und neu starten.

IP-Adresse festlegen: 10.x.1.21/24, Gateway 10.x.1.1.

DNS-Serveradresse auf 10.x.1.21 setzen.

(Für das Labor) Windows-Firewall vorübergehend deaktivieren.

Mit ipconfig /all und ping die Einstellungen prüfen.

Zweite Festplatte einbinden

Server Manager → File and Storage Services öffnen.

Neuen Storage Pool erstellen, 10-GB-Disk zuweisen (z. B. Name „Pool1“).

Virtuelle Disk anlegen, Layout „Simple“, Provisioning „Fixed“.

Volume erstellen, Laufwerksbuchstabe E, NTFS-Format.

Im Explorer den Ordner E:\AD anlegen.

Active-Directory-Rolle installieren

Server Manager → Manage → Add Roles and Features.

Active Directory Domain Services auswählen, „Add Features“ bestätigen.

Installation durchführen.

Vor dem Schließen Konfiguration exportieren nach E:\AD\DeploymentConfigTemplate.xml.

Hochstufung zum Domänencontroller

Im Server Manager auf Promote this server to a domain controller klicken.

Add a new forest wählen.

Root domain name: wondertoys.local.

Forest und Domain functional level: Windows Server 2016.

DNS Server anhaken, Global Catalog aktiviert lassen, kein RODC.

DSRM-Kennwort setzen.

Pfade anpassen:

Database und Log: E:\AD\NTDS

SYSVOL: E:\AD\SYSVOL

Hinweis zu „Parent Zone local“ ignorieren.

Installation starten und Server neu starten.

Danach Anmeldung mit WONDERTOYS\Administrator.

Überprüfung von AD und DNS

Prüfen, dass die Netzwerkkarte weiterhin DNS 10.x.1.21 verwendet.

DNS-Manager öffnen → Zone wondertoys.local:

Typ: Active Directory-integrated

Dynamic updates: Secure only

Unter _tcp müssen SRV-Einträge wie _gc, _kerberos, _ldap vorhanden sein.

Namensauflösung testen:

nslookup nyw9dc01

nslookup nyw9dc01.wondertoys.local

nslookup wondertoys.local

FSMO-Rollen und Global Catalog

In der Eingabeaufforderung netdom query fsmo ausführen.
Erwartung: Alle Rollen liegen auf NYW9DC01.

Optional:

Get-ADForest

Get-ADDomain
zur Kontrolle der Global-Catalog- und FSMO-Rollen.
