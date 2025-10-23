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


M159
Anleitung: Einrichtung der Subdomäne „work.wondertoys.local“

Ziel
In dieser Aufgabe wird eine untergeordnete Active-Directory-Subdomäne mit integriertem DNS-Server erstellt. Sie soll in die bestehende Domänenstruktur „wondertoys.local“ integriert werden. Die Subdomäne trägt den Namen „work.wondertoys.local“ und wird auf einem eigenen Domänencontroller eingerichtet. Zusätzlich wird auf dem Parent-Server eine DNS-Delegation für den neuen Namensraum eingerichtet.

Planung
Die bestehende Umgebung besteht aus der Domäne wondertoys.local mit dem Domänencontroller NYW9DC01. Dieser Server verwendet die IP-Adresse 10.3.1.21. Der neue Domänencontroller für die Subdomäne wird NYW9DC04 heißen und die IP-Adresse 10.3.1.24 erhalten. Beide Server befinden sich im selben Subnetz 10.3.1.0/24 mit dem Gateway 10.3.1.1. Während der Installation wird zunächst der DNS-Server des Parent-Controllers (10.3.1.21) verwendet. Nach der Installation zeigt der neue Server auf seinen eigenen DNS-Dienst. Die Funktionsebene wird auf Windows Server 2016 festgelegt. Installiert wird, wie bei der Stammdomäne, auf Laufwerk E: im Verzeichnis E:\AD. Dieses liegt auf einer zweiten virtuellen Festplatte mit einer Größe von 10 GB.

Vorbereitung der virtuellen Maschine
In VMware Player wird eine neue virtuelle Maschine erstellt. Es wird eine zweite virtuelle Festplatte mit 10 GB hinzugefügt. Dabei wird der Typ SCSI gewählt und die Option „Split into multiple files“ aktiviert. Das CD-Laufwerk wird vom physischen Laufwerk abgekoppelt, und die Soundkarte wird entfernt. Danach wird Windows Server 2019 installiert und vollständig aktualisiert.

Grundkonfiguration
Nach der Installation wird der Computername auf NYW9DC04 geändert. Danach erfolgt ein Neustart. Anschließend wird die Netzwerkkonfiguration festgelegt: IP-Adresse 10.3.1.24, Subnetzmaske 255.255.255.0, Gateway 10.3.1.1 und DNS-Server 10.3.1.21. Im Labor kann die Windows-Firewall vorübergehend deaktiviert werden. Mit den Befehlen „ipconfig /all“ und „ping“ auf 10.3.1.21 oder NYW9DC01.wondertoys.local wird die Erreichbarkeit geprüft.

Einbinden der zweiten Festplatte
Im Server Manager wird unter File and Storage Services ein neuer Storage Pool erstellt, dem die zweite Festplatte zugewiesen wird. Aus diesem Pool wird eine virtuelle Disk mit dem Layout „Simple“ und der Option „Fixed“ erstellt. Anschließend wird daraus ein neues Volume erzeugt, dem der Laufwerksbuchstabe E: zugewiesen wird. Das Dateisystem ist NTFS. Auf diesem Laufwerk wird der Ordner E:\AD erstellt.

Installation von Active Directory Domain Services
Im Server Manager wird unter „Manage“ die Option „Add Roles and Features“ gewählt. Dort wird die Rolle Active Directory Domain Services hinzugefügt. Nach der Installation wird die Konfiguration in die Datei E:\AD\DeploymentConfigTemplate.xml exportiert.

Hochstufung zum Domänencontroller
Nach der Installation wird im Server Manager auf „Promote this server to a domain controller“ geklickt. Es wird die Option „Add a new domain to an existing forest“ ausgewählt. Die bestehende Parent-Domäne lautet wondertoys.local, die neue Subdomäne wird work heißen, also work.wondertoys.local. Als Domain Type wird Child Domain gewählt. Die Funktionsebene bleibt Windows Server 2016. Der DNS-Server und der Global Catalog bleiben aktiviert, RODC wird nicht verwendet. Als Anmeldeinformationen wird der Benutzer WONDERTOYS\Administrator angegeben. Danach wird das DSRM-Kennwort gesetzt. Die Datenbank- und Logdateien werden auf E:\AD\NTDS und der SYSVOL-Ordner auf E:\AD\SYSVOL gespeichert. Anschließend wird die Installation gestartet und der Server neu gestartet.

DNS-Konfiguration auf dem neuen Domänencontroller
Nach dem Neustart wird überprüft, ob der Parent-Server aufgelöst werden kann. Dies geschieht mit „nslookup nyw9dc01.wondertoys.local“. Im DNS-Manager ist nun die Zone work.wondertoys.local sichtbar. Sie ist Active-Directory-integriert und erlaubt sichere dynamische Updates. Im nächsten Schritt wird eine unbedingte Weiterleitung zum Parent-DNS eingerichtet. Dazu wird im DNS-Manager des neuen Servers unter den Eigenschaften die IP-Adresse 10.3.1.21 als Forwarder hinzugefügt. Dadurch werden alle nicht lokal auflösbaren Anfragen an den Parent weitergeleitet. Nach Abschluss dieser Konfiguration wird die DNS-Serveradresse in den Netzwerkeinstellungen auf die eigene IP 10.3.1.24 geändert.

DNS-Delegierung auf dem Parent-Server
Auf NYW9DC01 wird im DNS-Manager in der Zone wondertoys.local eine neue Delegierung erstellt. Der Name lautet „work“. Als Nameserver wird nyw9dc04.work.wondertoys.local mit der IP-Adresse 10.3.1.24 angegeben. Nach dem Anlegen sollte der Delegationseintrag korrekt in der Zone sichtbar sein.

Rückwärtsauflösung
Auf dem Parent-Server NYW9DC01 wird zusätzlich eine Zone für die Rückwärtsauflösung eingerichtet. Dazu wird im DNS-Manager eine neue primäre Zone mit der Netzwerk-ID 10.3.1 erstellt. Die Zone trägt den Namen 1.3.10.in-addr.arpa. Dort werden Einträge für 10.3.1.21 (nyw9dc01.wondertoys.local) und 10.3.1.24 (nyw9dc04.work.wondertoys.local) hinzugefügt. Danach wird der DNS-Server neu gestartet.

Überprüfung
Die Namensauflösung wird getestet mit den Befehlen nslookup nyw9dc01.wondertoys.local, nslookup nyw9dc04.work.wondertoys.local und nslookup work.wondertoys.local. Zusätzlich können beide Server gegenseitig angepingt werden. Mit dem Befehl „netdom query fsmo“ wird überprüft, dass alle FSMO-Rollen weiterhin auf dem Parent-Domänencontroller liegen. Danach sollte getestet werden, ob Benutzer aus der Parent-Domäne auf Ressourcen der Subdomäne zugreifen können. Bei DNS-Problemen kann eine manuelle Replikation oder ein Neustart beider Server helfen.

Ergebnis
Nach der erfolgreichen Installation besteht die Stammdomäne wondertoys.local mit dem Domänencontroller NYW9DC01 und die Subdomäne work.wondertoys.local mit dem Domänencontroller NYW9DC04. Die DNS-Delegation funktioniert in beide Richtungen, die Auflösung ist getestet, und die Struktur ist vollständig integriert.

PowerShell-Aufgabe
Auf dem Domänencontroller der Subdomäne sollen per PowerShell Objekte im Active Directory erstellt werden. Dazu wird im Verzeichnis E:\AD die Datei ITObjektErzeugen.ps1 angelegt. Im Windows Explorer sollten Dateierweiterungen sichtbar sein. Die Datei wird mit Rechtsklick und „Edit“ in der PowerShell ISE geöffnet.

Im Skript werden nacheinander eine globale Sicherheitsgruppe, eine lokale Sicherheitsgruppe und ein Benutzer erstellt. Das Skript kann beispielsweise wie folgt aussehen:

# ITObjektErzeugen.ps1
# Erstellung von Gruppen und Benutzer in der Domäne work.wondertoys.local

New-ADGroup -Name "VerkaufsGruppeGlobal" -GroupScope Global -GroupCategory Security -Path "OU=Users,DC=work,DC=wondertoys,DC=local"

New-ADGroup -Name "ProduktOrdnerLesenGruppeLokal" -GroupScope DomainLocal -GroupCategory Security -Path "OU=Users,DC=work,DC=wondertoys,DC=local"

Add-ADGroupMember -Identity "ProduktOrdnerLesenGruppeLokal" -Members "VerkaufsGruppeGlobal"

New-ADUser -Name "Valentin Hefti" -GivenName "Valentin" -Surname "Hefti" -SamAccountName "vhefti" -UserPrincipalName "vhefti@work.wondertoys.local" -AccountPassword (ConvertTo-SecureString "P@ssword123" -AsPlainText -Force) -Enabled $true -Path "OU=Users,DC=work,DC=wondertoys,DC=local"

Add-ADGroupMember -Identity "ProduktOrdnerLesenGruppeLokal" -Members "vhefti"

Read-Host "Skript beendet – Enter zum Schließen"


Wenn beim Ausführen des Skripts die Meldung erscheint, dass das Ausführen von Skripten deaktiviert ist, muss die Berechtigung mit folgendem Befehl angepasst werden:

Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser


Das Skript wird mit F5 ausgeführt. Danach wird im Active Directory überprüft, ob die Gruppen und der Benutzer korrekt angelegt wurden. In der Gruppe ProduktOrdnerLesenGruppeLokal sollte sowohl die globale Gruppe VerkaufsGruppeGlobal als auch der Benutzer Valentin Hefti Mitglied sein.

Abschließend sollte getestet werden, ob das Skript fehlerfrei läuft und die Objekte wie gewünscht erzeugt. Damit ist die Subdomäne vollständig eingerichtet und betriebsbereit.
