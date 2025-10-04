# Lazzycorp - HackMyVM (Easy)
 ![Lazzycorp.png](Lazzycorp.png)

## Übersicht

*   **VM:** Lazzycorp
*   **Plattform:** https://hackmyvm.eu/machines/machine.php?vm=Lazzycorp
*   **Schwierigkeit:** Easy
*   **Autor der VM:** DarkSpirit
*   **Datum des Writeups:** 20. August 2025
*   **Original-Writeup:** https://alientec1908.github.io/Lazzycorp_HackMyVM_Easy/
*   **Autor:** Ben C.

## Kurzbeschreibung

Die Challenge "Lazzycorp" ist eine "Easy"-Maschine, die mehrere klassische Pentesting-Techniken kombiniert. Der Angriff beginnt mit der Enumeration eines anonymen FTP-Servers, um eine Bilddatei zu finden. Mittels Steganographie werden aus diesem Bild Anmeldedaten für ein verstecktes Web-Panel extrahiert. Der initiale Zugriff wird durch eine ungesicherte Dateiupload-Funktion in diesem Panel erreicht, was zu einer Reverse Shell führt. Die Rechteausweitung zu `root` erfolgt durch das Ausnutzen eines fehlerhaften SUID-Binaries, das für einen PATH-Hijacking-Angriff anfällig ist.

## Disclaimer / Wichtiger Hinweis

Die in diesem Writeup beschriebenen Techniken und Werkzeuge dienen ausschließlich zu Bildungszwecken im Rahmen von legalen Capture-The-Flag (CTF)-Wettbewerben und Penetrationstests auf Systemen, für die eine ausdrückliche Genehmigung vorliegt. Die Anwendung dieser Methoden auf Systeme ohne Erlaubnis ist illegal. Der Autor übernimmt keine Verantwortung für missbräuchliche Verwendung der hier geteilten Informationen. Handeln Sie stets ethisch und verantwortungsbewusst.

## Verwendete Tools

*   `nmap`
*   `nikto`
*   `gobuster`
*   `feroxbuster`
*   `ftp`
*   `stegseek`
*   `curl`
*   `netcat`
*   `ssh`
*   Standard Linux-Befehle (`ls`, `cat`, `find`, `echo`, `chmod`, `export`, etc.)

## Lösungsweg (Zusammenfassung)

Der Angriff auf die Maschine "Lazzycorp" gliederte sich in folgende Phasen:

1.  **Reconnaissance & Enumeration:**
    *   Ein `nmap`-Scan identifizierte die offenen Ports 21 (FTP), 22 (SSH) und 80 (HTTP). Der FTP-Server erlaubte einen anonymen Login, was zur Entdeckung des Verzeichnisses `/pub` und der Datei `note.jpg` führte. Auf der Webseite wurden über `robots.txt` und `gobuster` mehrere Verzeichnisse, darunter `/blog` und ein verstecktes Login-Panel unter `/auth-lazycorp-dev/`, aufgedeckt.

2.  **Initial Access (Steganography & File Upload RCE):**
    *   Die heruntergeladene Datei `note.jpg` wurde mit `stegseek` und der `rockyou.txt`-Wortliste analysiert. Dies extrahierte eine versteckte Datei `creds.txt` mit den Anmeldedaten `dev:d3v3l0pm3nt!nt3rn`.
    *   Diese Anmeldedaten funktionierten für das zuvor entdeckte Panel unter `/auth-lazycorp-dev/login.php`.
    *   Nach dem Login wurde eine Dateiupload-Funktion ausgenutzt, um eine einfache PHP-Reverse-Shell hochzuladen. Durch den Aufruf der hochgeladenen Datei wurde eine Verbindung zu einem `netcat`-Listener hergestellt, was eine Shell als Benutzer `www-data` gewährte.

3.  **Post-Exploitation / Privilege Escalation (von `www-data` zu `arvind`):**
    *   Als `www-data` wurde das System weiter enumeriert. Im Home-Verzeichnis `/home/arvind` wurde ein lesbarer SSH Private Key (`.ssh/id_rsa`) gefunden.
    *   Mit diesem privaten Schlüssel konnte eine SSH-Verbindung als Benutzer `arvind` hergestellt werden, was einen stabileren und höher privilegierten Zugriff ermöglichte. Die User-Flag wurde ebenfalls in diesem Verzeichnis gefunden.

4.  **Privilege Escalation (von `arvind` zu `root`):**
    *   Als `arvind` wurde im Home-Verzeichnis ein SUID-Binary namens `reset` entdeckt. Eine `strings`-Analyse der Datei offenbarte, dass dieses Binary das Skript `/usr/bin/reset_site.sh` ausführt.
    *   Die Berechtigungen des Skripts zeigten, dass es dem Benutzer `arvind` gehört und von ihm beschreibbar ist. Das Skript selbst verwendete Befehle wie `cp` und `rm` ohne absolute Pfade (z.B. `/bin/cp`).
    *   Dies ermöglichte einen PATH-Hijacking-Angriff: Im Verzeichnis `/tmp` wurde eine bösartige Datei namens `cp` erstellt, die `/bin/bash` ausführt. Das `/tmp`-Verzeichnis wurde an den Anfang der `$PATH`-Umgebungsvariable gesetzt.
    *   Durch Ausführen des SUID-Binaries `./reset` wurde nun anstelle des echten `cp`-Befehls die bösartige Datei in `/tmp` mit Root-Rechten ausgeführt, was zu einer sofortigen Root-Shell führte.

## Wichtige Schwachstellen und Konzepte

*   **Steganographie:** Anmeldeinformationen waren mittels `stegseek` in einer Bilddatei versteckt. Dies unterstreicht die Notwendigkeit, heruntergeladene Artefakte auf versteckte Daten zu untersuchen.
*   **Unrestricted File Upload:** Eine klassische Web-Schwachstelle, bei der der Server den Typ der hochgeladenen Dateien nicht validiert, was die Ausführung von serverseitigem Code (hier PHP) ermöglicht.
*   **SUID Binary & PATH Hijacking:** Ein benutzerdefiniertes SUID-Binary, das externe Skripte oder Befehle ohne absolute Pfade aufruft, ist ein kritischer Fehler. Dies erlaubte die Manipulation des `$PATH`, um einen bösartigen Befehl mit den Rechten des SUID-Besitzers (root) auszuführen.
*   **Information Disclosure:** Sensible Informationen, wie die Existenz eines SSH-Schlüssels oder Hinweise in Blog-Posts, erleichterten die laterale Bewegung und Eskalation erheblich.

## Flags

*   **User Flag (`/home/arvind/user.txt`):** `FLAG{you_got_foothold_nice}`
*   **Root Flag (`/root/root.txt`):** `FLAG{lazycorp_reset_exploit_worked}`

## Tags

`HackMyVM`, `Lazzycorp`, `Easy`, `Steganography`, `FileUpload`, `SUID`, `PATHHijacking`, `Linux`, `Web`, `Privilege Escalation`, `FTP`
