# Raspberry Pi einrichten und absichern

Ziel: Einen Raspberry Pi als Heimlabor aufsetzen, per SSH erreichbar machen und grundlegend absichern, als Basis für Projekte wie Pi-hole.

## Material
- Raspberry Pi 5
- microSD-Karte (SanDisk, in Windows als 59 GB angezeigt)
- Netzteil, Anbindung per WLAN
- PC mit Windows 11 und Kartenleser

## Umsetzung

### 1. Betriebssystem auf die SD-Karte schreiben
- Programm: Raspberry Pi Imager
- System: Raspberry Pi OS Lite (64-bit), basiert auf Debian 13 "trixie"
- Im Imager vorab eingestellt: Hostname, Benutzername, Passwort, WLAN, SSH aktiviert
- "Lite" bedeutet: ohne grafische Oberfläche, der Pi wird nur per Terminal bedient.

### 2. Per SSH verbinden
```
ssh benutzername@hostname.local
```
- `ssh` baut eine verschlüsselte Verbindung zum Pi auf.

### 3. System aktualisieren
```
sudo apt update && sudo apt upgrade -y
```
- `apt update` lädt die Liste der verfügbaren Software-Versionen, `apt upgrade` installiert die neuen Versionen (`-y` beantwortet Rückfragen mit Ja).
- Updates schließen bekannte Sicherheitslücken.

### 4. Firewall einrichten
```
sudo apt install ufw -y
sudo ufw allow ssh
sudo ufw enable
sudo ufw status
```
- `ufw` ist eine einfache Bedienung der Linux-Firewall.
- `allow ssh` erlaubt Port 22. Das muss **vor** `enable` kommen, sonst sperrt man sich selbst aus.
- Alles Eingehende, was nicht erlaubt ist, wird blockiert (Default deny).

### 5. SSH-Schlüssel statt Passwort
Auf dem PC (PowerShell):
```
ssh-keygen -t ed25519 -C "homelab-pi"
```
- Erzeugt ein Schlüsselpaar (privat und öffentlich) mit dem Verfahren Ed25519, geschützt durch eine Passphrase.

Den öffentlichen Schlüssel auf den Pi übertragen:
```
type $env:USERPROFILE\.ssh\id_ed25519.pub | ssh benutzername@hostname.local "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"
```
- Die Pipe `|` gibt den Schlüssel an `ssh` weiter, und der Befehl auf dem Pi hängt ihn an `authorized_keys` an.

Nach einem erfolgreichen Test des Logins mit Schlüssel habe ich auf dem Pi den Passwort-Login abgeschaltet. In `/etc/ssh/sshd_config.d/00-hardening.conf`:
```
PasswordAuthentication no
KbdInteractiveAuthentication no
```
Danach prüfen und neu starten:
```
sudo sshd -t
sudo systemctl restart ssh
sudo sshd -T | grep -i -E "passwordauthentication|kbdinteractiveauthentication"
```
- `sshd -t` prüft die Konfiguration auf Fehler, `systemctl restart ssh` startet den SSH-Server neu, `sshd -T` zeigt die wirksamen Einstellungen. Beide Zeilen müssen `no` zeigen.
- Der private Schlüssel liegt nur bei mir (Backup auf USB-Stick), die Passphrase im Passwortmanager.

### 6. Pi herunterfahren
```
sudo shutdown -h now
```
- Fährt den Pi sauber herunter (`-h` = anhalten, `now` = sofort), damit die SD-Karte keinen Schaden nimmt.
- Sehr Praktisch auch zum schlafen gehen, damit das Licht ausgeht
## Probleme und Lösungen

### Imager: "Error writing to storage device"
- **Problem:** Der Imager brach beim Vorbereiten der Karte ab. Die Karte wurde im Explorer nicht angezeigt.
- **Ursache:** In der Datenträgerverwaltung stand sie als **RAW** (ohne Dateisystem).
- **Lösung:** Als exFAT formatiert, Laufwerksbuchstaben hinzugefügt und die Karte mit `diskpart` und `clean` komplett zurückgesetzt. Danach lief der Imager durch.
- **Gelernt:** Was ein Dateisystem ist und wie man mit der Datenträgerverwaltung und `diskpart` Speichermedien untersucht und zurücksetzt.

### Zugangstoken in einer Konfigurationsdatei
- Beim Umsehen im Home-Ordner (`ls -la`, `cat`) fand ich in `~/.config/` die Konfigurationsdatei eines Fernzugriffsdienstes (Raspberry Pi Connect) mit einem Zugangstoken.
- **Gelernt:** Konfigurationsdateien können Geheimnisse enthalten. Solche Dateien nie hochladen oder zeigen, und ungenutzte Fernzugriffsdienste ausschalten.

## Was ich gelernt habe
- `sudo` führt einzelne Befehle mit Administratorrechten aus.
- Ein Passwort wird zum Server geschickt und kann erraten werden. Ein SSH-Schlüssel braucht eine Datei plus Passphrase, die lokal bleibt.
- Rechte in `ls -la` (`drwx------`) zeigen, wer lesen, schreiben und ausführen darf. `.ssh` ist absichtlich nur für den Besitzer zugänglich.
- Eine Firewall sollte standardmäßig alles blockieren und nur Nötiges erlauben.
- Einen alternativen DNS-Server aufstellen um immernoch WLAN zu haben, nachdem ich mein Pi ausmache.

## Hinweis
Adressen, Passwörter, Tokens und Schlüssel sind in diesem Repo bewusst nicht enthalten.

### Blocklisten erweitert
Zusätzlich zur Standardliste habe ich [HaGeZi Multi Pro / weitere Liste eintragen] hinzugefügt. Listen gehören in die Weboberfläche unter **Adlists** (nicht in die Allowlist). Danach `sudo pihole -g` ("gravity" lädt alle Listen herunter und führt sie zusammen). Mehr Listen bringen nicht automatisch mehr: Viele überschneiden sich, und je strenger sie sind, desto öfter geht eine Seite kaputt. Fehlblockierte Domains gebe ich mit `sudo pihole allow domain.de` frei.
