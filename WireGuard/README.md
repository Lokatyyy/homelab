## 24. Oktober 2026
**WireGuard-VPN auf dem Pi eingerichtet**

- Nach dem Neustart des Pi geprüft, ob alles von selbst wieder läuft: VPN, Pi-hole, SSH und Firewall waren aktiv.
- **Problem:** Mein altes VPN funktionierte nach dem Anbieterwechsel nicht mehr. In der FRITZ!Box steht, dass der Anschluss einen DS-Lite-Tunnel nutzt. Das heißt, ich habe keine eigene öffentliche IPv4-Adresse und kann über IPv4 keine Ports freigeben. Der Weg ins Heimnetz geht nur über IPv6.
- WireGuard unter Debian 13 von Hand installiert (`apt install wireguard`), je ein Schlüsselpaar für den Pi und für das Handy erzeugt und die Server-Konfiguration `wg0.conf` geschrieben.
  - **Fehler:** Der Dienst startete nicht, weil ich in `wg0.conf` die Platzhalter für die Schlüssel nicht ersetzt hatte. Die Ursache fand ich im Protokoll mit `journalctl`.
- Firewall `ufw`: UDP-Port 51820 für den VPN erlaubt, DNS-Anfragen (Port 53) nur aus dem VPN-Netz an Pi-hole.
- FRITZ!Box: Portfreigabe für UDP 51820 nur über IPv6, ohne "exposed host", damit nur dieser eine Port offen ist. Die Menüs waren auf Französisch, und ich habe zuerst die falsche Maske (MyFRITZ!) erwischt.
- DuckDNS: Name angelegt, damit das Handy den Pi auch findet, wenn sich mein IPv6-Präfix ändert. Ein Skript meldet die Adresse alle 5 Minuten per `cron`. Den Token habe ich in eine eigene Datei mit `chmod 600` gelegt, damit er nicht in der Befehlsliste (`.bash_history`) landet.
  - **Fehler:** Das Update lieferte `KO`. Gefunden habe ich es mit Tests: Länge des Tokens prüfen, versteckte Zeichen mit `cat -A` sichtbar machen und die Anfrage ohne IP-Adresse testen. Ipv6 auf DuuckDNS nciht eingetragen und Token nicht richtig eingefügt
- Handy: Konfiguration per QR-Code in die WireGuard-App übernommen. Handshake ja
- **Gelernt:**
  - DS-Lite heißt: keine eigene IPv4, Zugang von außen nur über IPv6.
  - Bei WireGuard hat jedes Gerät ein Schlüsselpaar. Der private Schlüssel bleibt beim Gerät, nur der öffentliche wird getauscht.
  - Ein DDNS-Name bleibt gleich, auch wenn sich die Adresse dahinter ändert.
  - Geheimnisse (Token, private Schlüssel) gehören in geschützte Dateien, nicht in Befehle und nicht ins Repo.
- **Offen:** Ändert sich mein IPv6-Präfix, und aktualisiert das Skript den Eintrag dann zuverlässig? Wie sieht ein Vollzugriff auf das Heimnetz statt nur DNS über den VPN aus?

## 25. Oktober 2026
- Pi-hole um zusätzliche Blocklisten erweitert (StevenBlack und HaGeZi) und mit `pihole -g` neu geladen.
- **Gelernt:** Mehr Listen heißt nicht automatisch besser, weil sich viele überschneiden und Seiten kaputtgehen können. `sudo wg show` ; `pihole status` beim anschalten des Pis zum checken ob alles läuft
