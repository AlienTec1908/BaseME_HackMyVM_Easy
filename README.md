# BaseME - HackMyVM Writeup

![BaseME VM Icon](BaseME.png)

Dieses Repository enthält das Writeup für die HackMyVM-Maschine "BaseME" (Schwierigkeitsgrad: Easy), erstellt von DarkSpirit. Ziel war es, initialen Zugriff auf die virtuelle Maschine zu erlangen und die Berechtigungen bis zum Root-Benutzer zu eskalieren.

## VM-Informationen

*   **VM Name:** BaseME
*   **Plattform:** HackMyVM
*   **Autor der VM:** DarkSpirit
*   **Schwierigkeitsgrad:** Easy
*   **Link zur VM:** [https://hackmyvm.eu/machines/machine.php?vm=BaseME](https://hackmyvm.eu/machines/machine.php?vm=BaseME)

## Writeup-Informationen

*   **Autor des Writeups:** Ben C.
*   **Datum des Berichts:** 20. Oktober 2022
*   **Link zum Original-Writeup (GitHub Pages):** [https://alientec1908.github.io/BaseME_HackMyVM_Easy/](https://alientec1908.github.io/BaseME_HackMyVM_Easy/)

## Kurzübersicht des Angriffspfads

Der Angriff auf die BaseME-Maschine konzentrierte sich stark auf Base64-kodierte Informationen:

1.  **Reconnaissance:**
    *   Identifizierung der Ziel-IP (`192.168.2.100`) mittels `arp-scan`.
    *   Ein `nmap`-Scan offenbarte zwei offene Ports: SSH (22, OpenSSH 7.9p1) und HTTP (80, Nginx 1.14.2).
2.  **Web Enumeration & Base64 Clues:**
    *   `gobuster` fand nur eine `/index.html`-Datei.
    *   Der Inhalt von `/index.html` (oder eine darin gefundene Base64-kodierte Nachricht) enthielt einen Hinweis von "lucas": "ALL, absolutely ALL that you need is in BASE64. Including the password that you need :)".
    *   Mehrere Base64-kodierte Strings wurden gefunden und dekodiert (z.B. `aWxvdmV5b3UK` -> `iloveyou`), was zu einer initialen Passwortliste führte. Ein SSH-Brute-Force-Versuch mit `hydra` und diesen Passwörtern auf den Benutzer `lucas` schlug jedoch fehl.
    *   Eine `for`-Schleife wurde verwendet, um gängige Verzeichnis-/Dateinamen aus `common.txt` Base64 zu kodieren und in `dic.txt` zu speichern.
    *   `wfuzz` wurde mit `dic.txt` verwendet, um nach Base64-kodierten Pfaden auf dem Webserver zu suchen. Dies führte zur Entdeckung von:
        *   `/aWRfcnNhCg` (dekodiert: `id_rsa\n`)
        *   `/cm9ib3RzLnR4dAo=` (dekodiert: `robots.txt\n` - enthielt "Nothing here :(")
    *   Der Inhalt von `/aWRfcnNhCg` war ein weiterer Base64-kodierter Block, der nach Dekodierung einen passwortgeschützten privaten SSH-Schlüssel offenbarte.
3.  **Initial Access:**
    *   Der extrahierte SSH-Schlüssel war passwortgeschützt.
    *   `ssh2john` wurde verwendet, um den Hash der Passphrase aus der Schlüsseldatei zu extrahieren.
    *   `john` knackte den Hash mit `rockyou.txt` und fand die Passphrase `iloveyou` (eine der ursprünglich Base64-dekodierten Phrasen).
    *   Erfolgreicher SSH-Login als Benutzer `lucas` mit dem privaten Schlüssel und der geknackten Passphrase.
    *   Die User-Flag (`/home/lucas/user.txt`) wurde gelesen: `HMV8nnJAJAJA`.
4.  **Privilege Escalation:**
    *   `sudo -l` für `lucas` zeigte: `(ALL) NOPASSWD: /usr/bin/base64`.
    *   Diese Berechtigung wurde ausgenutzt, um die Root-Flag zu lesen: `sudo /usr/bin/base64 /root/root.txt | base64 -d`.
5.  **Flags:**
    *   Die User-Flag wurde als `lucas` gelesen.
    *   Die Root-Flag wurde mittels der `sudo base64`-Berechtigung gelesen: `HMVFKBS64`.

## Verwendete Tools

*   `arp-scan`
*   `nmap`
*   `gobuster`
*   `echo`
*   `base64`
*   `hydra`
*   `wfuzz`
*   `nikto`
*   `ssh-keyscan`
*   `bash` (for loop)
*   `vi`
*   `chmod`
*   `ssh`
*   `ssh2john`
*   `john`
*   `cat`
*   `ls`
*   `sudo`
*   `id`

## Identifizierte Schwachstellen (Zusammenfassung)

*   **Preisgabe von Hinweisen durch Base64-Kodierung:** Ein Hinweis im Webseiteninhalt lenkte die Aufmerksamkeit auf Base64.
*   **Verstecken von Ressourcen durch Base64-kodierte Pfadnamen:** Pfade zu sensiblen Dateien (privater SSH-Schlüssel) waren Base64-kodiert.
*   **Verfügbarkeit eines privaten SSH-Schlüssels über HTTP:** Ein privater SSH-Schlüssel war, wenn auch doppelt Base64-kodiert, über den Webserver zugänglich.
*   **Schwache SSH-Schlüssel-Passphrase:** Die Passphrase des privaten Schlüssels (`iloveyou`) war in gängigen Wortlisten enthalten.
*   **Unsichere `sudo`-Konfiguration:** Der Benutzer `lucas` konnte `/usr/bin/base64` ohne Passwort als `root` ausführen, was das Lesen beliebiger Dateien als Root ermöglichte.

## Flags

*   **User Flag (`/home/lucas/user.txt`):** `HMV8nnJAJAJA`
*   **Root Flag (`/root/root.txt` via `sudo base64`):** `HMVFKBS64`

---

**Wichtiger Hinweis:** Dieses Dokument und die darin enthaltenen Informationen dienen ausschließlich zu Bildungs- und Forschungszwecken im Bereich der Cybersicherheit. Die beschriebenen Techniken und Werkzeuge sollten nur in legalen und autorisierten Umgebungen (z.B. eigenen Testlaboren, CTFs oder mit expliziter Genehmigung) angewendet werden. Das unbefugte Eindringen in fremde Computersysteme ist eine Straftat und kann rechtliche Konsequenzen nach sich ziehen.

---
*Bericht von Ben C. - Cyber Security Reports*
