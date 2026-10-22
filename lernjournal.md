## 15. Oktober 2026
- Erster Tag, GitHub-Account und TryHackMe-Account erstellt
- TryHackMe "Introduction to Cyber Security" und "Inside a Computer System" gemacht
- **Gelernt:** Grundlagen von Cyber Security (Introduction to Cyber Security) und wie ein Computer aufgebaut ist (Inside a Computer System). Dazu auch, was für Jobs es in Cybersecurity gibt

## 16. Oktober 2026
- Mit GitHub vertraut gemacht
- Dieses Repo erstellt
- OverTheWire Bandit Level 0-5 abgeschlossen
- **Gelernt:** Verbindung per SSH, `ls`, `ls -la` (versteckte Dateien), `cd`, `cat`, `pwd`
- **Schwierig:** Dateinamen mit `-` und mit Leerzeichen. Gelöst mit `./` davor bzw. Anführungszeichen

## 17. Oktober 2026
- OverTheWire Bandit Level 6-11 abgeschlossen
- **Gelernt:** `base64`, `tr 'A-Za-z' 'N-ZA-Mn-za-m'`, `strings`, `uniq`, `grep`, `find`, `|`
- **Schwierig:** Die Bedingungen von `find` zu wissen und `tr 'A-Za-z' 'N-ZA-Mn-za-m'` für ROT13. Verstanden habe ich: `tr` ersetzt die Zeichen der ersten Liste der Reihe nach durch die der zweiten, und 13 Stellen sind die Hälfte des Alphabets.

## 22. Oktober 2026
**Raspberry Pi eingerichtet und abgesichert**

- Raspberry Pi OS Lite (64-bit) mit dem Raspberry Pi Imager auf die SD-Karte geschrieben, Verbindung per SSH.
- **Problem:** Der Imager brach mit "Error writing to storage device" ab. Die Karte wurde im Explorer nicht angezeigt und stand in der Datenträgerverwaltung als RAW (ohne Dateisystem).
  - **Lösung:** Als exFAT formatiert, Laufwerksbuchstaben hinzugefügt und in PowerShell mit `diskpart` und `clean` zurückgesetzt.
- **Firewall:** `ufw` installiert, SSH (Port 22) erlaubt, danach aktiviert. Regel: alles Eingehende ist blockiert, außer es wird erlaubt (Default deny).
- **SSH-Schlüssel:** Schlüsselpaar mit `ssh-keygen -t ed25519` erzeugt, öffentlichen Schlüssel in `authorized_keys` auf dem Pi abgelegt, Login mit Schlüssel getestet, danach Passwort-Login in der SSH-Konfiguration ausgeschaltet.
- Privaten Schlüssel auf einem USB-Stick gesichert, Passphrase im Passwortmanager.
- **Gelernt:**
  - `sudo` führt einen einzelnen Befehl mit Administratorrechten aus.
  - Ein Passwort wird zum Server geschickt und kann erraten werden. Ein SSH-Schlüssel braucht eine Datei plus Passphrase, und die Passphrase bleibt lokal.
  - In `ls -la` zeigen die Rechte (`drwx------`) Besitzer, Gruppe und andere. `.ssh` ist absichtlich nur für mich lesbar.
  - Konfigurationsdateien können Geheimnisse enthalten: In `~/.config/` lag ein Zugangstoken von Raspberry Pi Connect. Solche Dateien nie hochladen oder zeigen.
- **Offen:** Ist Raspberry Pi Connect aktiv, und brauche ich es? Wie funktioniert `ssh-agent`?
