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



Aufgabe 3.2


Anleitung: Einrichtung der Subdomäne „work.wondertoys.local“
Ziel

Eine untergeordnete Active-Directory-Subdomäne mit integriertem DNS-Server wird in die bestehende Domänenstruktur von wondertoys.local integriert.
Die Subdomäne heißt work.wondertoys.local und wird auf einem separaten Domänencontroller eingerichtet.
Die DNS-Zone wird auf dem Parent-Server NYW9DC01 an die neue Subdomäne delegiert.

Planung
Komponente	Wert
Root-Domäne (Parent)	wondertoys.local
Subdomäne (Child)	work.wondertoys.local
Parent-Servername	NYW9DC01
Child-Servername	NYW9DC04
Funktionsebene	Windows Server 2016
Standort	New York
IP-Bereich	10.3.1.0/24
Gateway	10.3.1.1
Parent-DC (NYW9DC01)	10.3.1.21
Child-DC (NYW9DC04)	10.3.1.24
DNS-Server (vor Installation)	10.3.1.21 (Parent)
DNS-Server (nach Installation)	10.3.1.24 (eigener DNS)
Installationspfad	E:\AD
Volume-Größe	10 GB
Rollen	Active Directory Domain Services, DNS, Global Catalog
Vorbereitung der VM

In VMware Player eine neue virtuelle Maschine (WS5) für den Child-DC erstellen.

Zweite virtuelle Festplatte mit 10 GB hinzufügen:

Typ: SCSI

„Create a new virtual disk“

10 GB → „Split into multiple files“

CD-Laufwerk auf „Physical Drive“ abkoppeln, Soundkarte entfernen.

Windows Server 2019 installieren, alle Updates durchführen.

Grundkonfiguration des Servers

Hostname ändern:
Systemsteuerung → System → Computername → „Change“ →
Neuer Name: NYW9DC04
→ Neustart.

Netzwerk konfigurieren:

IP-Adresse:   10.3.1.24
Subnetzmaske: 255.255.255.0
Gateway:      10.3.1.1
DNS-Server:   10.3.1.21  (Parent)


(Für das Labor) Windows-Firewall vorübergehend deaktivieren.

Prüfung der Netzwerkkonfiguration:

ipconfig /all
ping 10.3.1.21
ping nyw9dc01.wondertoys.local

Zweite Festplatte einbinden

Server Manager → File and Storage Services → Storage Pools öffnen.

Neuen Storage Pool anlegen (z. B. Pool1) und 10-GB-Disk zuweisen.

Virtuelle Disk erstellen:

Layout: Simple

Provisioning: Fixed

Neues Volume erstellen:

Laufwerksbuchstabe: E:

Format: NTFS

Ordner anlegen:

E:\AD

Active Directory Domain Services installieren

Server Manager → Manage → Add Roles and Features.

Active Directory Domain Services auswählen → Add Features bestätigen.

Installation durchführen.

Vor dem Schließen die Konfiguration exportieren:

E:\AD\DeploymentConfigTemplate.xml

Hochstufung zum Domänencontroller (Child Domain)

Nach der Installation auf Promote this server to a domain controller klicken.

Add a new domain to an existing forest auswählen.

Parent domain name: wondertoys.local

New domain name: work
→ ergibt work.wondertoys.local

Domain Type: Child Domain

Functional Level: Windows Server 2016

DNS-Server aktivieren, Global Catalog aktiv lassen, kein RODC.

Anmeldeinformationen:
Benutzer: WONDERTOYS\Administrator
(Achtung: vollständigen FQDN angeben!)

DSRM-Kennwort setzen.

Pfadangaben:

Database: E:\AD\NTDS
Log files: E:\AD\NTDS
SYSVOL:   E:\AD\SYSVOL


Installation starten → Neustart.

DNS-Konfiguration auf dem Child-DC (NYW9DC04)

Nach dem Neustart prüfen:

nslookup nyw9dc01.wondertoys.local


→ Namensauflösung des Parent-DC muss funktionieren.

DNS-Manager öffnen

Zone work.wondertoys.local sollte automatisch erstellt sein.

Typ: Active Directory-integrated

Dynamic updates: Secure only

Unbedingte Weiterleitung einrichten:

Im DNS-Manager von NYW9DC04:

Rechtsklick auf Server → Properties → Forwarders

Hinzufügen: 10.3.1.21 (NYW9DC01)
→ Damit werden alle externen Anfragen an den Parent weitergeleitet.

DNS-Serveradresse anpassen:

Nach erfolgreicher AD-Installation wieder auf eigene IP (10.3.1.24) umstellen.

DNS-Delegierung auf dem Parent-DC (NYW9DC01)

Auf NYW9DC01 → DNS Manager → Zone wondertoys.local öffnen.

Rechtsklick → New Delegation

Name: work

FQDN des Nameservers: nyw9dc04.work.wondertoys.local

IP-Adresse: 10.3.1.24

Überprüfen, dass der Delegationseintrag korrekt erscheint.

Rückwärtsauflösungszone (Reverse Lookup Zone)

Auf NYW9DC01 im DNS-Manager:

New Zone → Primary Zone

Network ID: 10.3.1

Zone Name: 1.3.10.in-addr.arpa

FQDN-Einträge hinzufügen:

10.3.1.21 → nyw9dc01.wondertoys.local
10.3.1.24 → nyw9dc04.work.wondertoys.local


DNS-Server neu starten.

Tests und Überprüfung

Namensauflösung prüfen:

nslookup nyw9dc01.wondertoys.local
nslookup nyw9dc04.work.wondertoys.local
nslookup work.wondertoys.local


Ping-Test:

ping nyw9dc01
ping nyw9dc04


Active Directory testen:

netdom query fsmo


→ FSMO-Rollen bleiben auf NYW9DC01.

Überprüfen, dass Benutzer aus wondertoys.local sich in work.wondertoys.local anmelden können.

Replikationszeit beachten (DNS-Einträge evtl. manuell aktualisieren oder Server neu starten).

Ergebnis

Nach erfolgreicher Einrichtung:

Parent-Domäne: wondertoys.local

Subdomäne: work.wondertoys.local

Parent-DC: NYW9DC01 (10.3.1.21)

Child-DC: NYW9DC04 (10.3.1.24)

DNS-Delegation erfolgreich eingerichtet

Auflösung in beide Richtungen funktioniert

SYSVOL & NTDS auf separatem Laufwerk (E:)

🧩 Aufgabe 2 – PowerShell-Befehle mit AD Administrative Center
Ziel

In der Domäne work.wondertoys.local sollen mit PowerShell und dem AD Administrative Center
neue Gruppen und Benutzerobjekte erstellt werden.

Vorbereitung

Anmeldung auf dem NYW9DC04 (Subdomäne work.wondertoys.local).

Im Explorer Dateinamenerweiterungen aktivieren:

View → Options → View → Haken bei Hide extensions for known file types entfernen.

Im Ordner E:\AD eine neue Datei erstellen:

ITObjektErzeugen.ps1

PowerShell-Skript editieren

Rechtsklick → Edit → öffnet sich in Windows PowerShell ISE.

Kommentare mit # beginnen.

Am Ende ggf. ein Read-Host einfügen, damit das Fenster offen bleibt.

Skriptinhalt (Beispiel ITObjektErzeugen.ps1)
# ITObjektErzeugen.ps1
# Autor: [Dein Name]
# Domäne: work.wondertoys.local
# Ziel: Erstellung von Gruppen und Benutzerobjekten im AD

# Globale Sicherheitsgruppe erstellen
New-ADGroup -Name "VerkaufsGruppeGlobal" -GroupScope Global -GroupCategory Security -Path "OU=Users,DC=work,DC=wondertoys,DC=local" -Description "Globale Verkaufsgruppe"

# Lokale Sicherheitsgruppe erstellen
New-ADGroup -Name "ProduktOrdnerLesenGruppeLokal" -GroupScope DomainLocal -GroupCategory Security -Path "OU=Users,DC=work,DC=wondertoys,DC=local" -Description "Lokale Gruppe für Lesezugriff auf Produktordner"

# Globale Gruppe als Mitglied hinzufügen
Add-ADGroupMember -Identity "ProduktOrdnerLesenGruppeLokal" -Members "VerkaufsGruppeGlobal"

# Benutzer anlegen
New-ADUser -Name "Valentin Hefti" -GivenName "Valentin" -Surname "Hefti" -SamAccountName "vhefti" -UserPrincipalName "vhefti@work.wondertoys.local" -AccountPassword (ConvertTo-SecureString "P@ssword123" -AsPlainText -Force) -Enabled $true -Path "OU=Users,DC=work,DC=wondertoys,DC=local"

# Benutzer in lokale Gruppe aufnehmen
Add-ADGroupMember -Identity "ProduktOrdnerLesenGruppeLokal" -Members "vhefti"

Read-Host "Skript beendet – Enter zum Schließen"

Wichtige Hinweise

Falls die Fehlermeldung "running scripts is disabled on this system" erscheint:

Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser


PowerShell-History im AD Administrative Center aktivieren:
„Show All“ → zeigt automatisch generierte Befehle bei manuellen Aktionen.

Tests und Kontrolle

Skript ausführen:

Rechtsklick → Run with PowerShell

Ausgabe prüfen.

Im AD Administrative Center oder ADUC:

VerkaufsGruppeGlobal

ProduktOrdnerLesenGruppeLokal

Valentin Hefti
→ sollten korrekt angelegt sein.

Mitgliedschaften prüfen:

Get-ADGroupMember "ProduktOrdnerLesenGruppeLokal"


Optional: Skript um Löschbefehle erweitern, um Tests rückgängig zu machen.

✅ Gesamtergebnis

Subdomäne work.wondertoys.local erfolgreich in wondertoys.local integriert.

DNS-Delegation funktioniert bidirektional.

PowerShell-Skript erstellt Gruppen und Benutzer korrekt.

Namensauflösung und Replikation geprüft.
