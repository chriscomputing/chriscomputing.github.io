+++
title = "Mein günstiges Homelab: Self-Hosting mit Debian, ZFS, und Docker"
description = "Kostengünstiges, energieeffizientes Homelab mit Debian Linux und ZFS für digitale Souveränität"
slug = "budget-homelab-debian-zfs-docker"
tags = ["Homelab", "Linux", "ZFS", "Selfhosting", "Monitoring","Docker"]
date = 2026-08-15
draft = false
# featured = true
math = false
toc_open = true
+++

Eine fortlaufende aktualisierte Übersicht was ich in meinem Homelab selbst hoste. Weil ich meine digitale Unabhängigkeit schätze fokussiere ich mich auf Anwendungen um meine Abhängigkeit von außer-europäischen Technologiekonzernen zu reduzieren. Dank Immich sind meine Bilder unter meiner Kontrolle (ein großartiger Step um mit de-googling anzufangen), und dank Paperless-ngx sind meine Dokumente hervorragend organisiert, und das ebenfalls vollständig unter meiner Kontrolle. Diese und weitere Anwendungen laufen auf meinem Debian 13 server welcher ZFS und Docker nutzt. Ziel meines Techstacks ist es, ein zuverlässiges System mit geringem Wartungsaufwand zu erschaffen.

## Pontus: Mein günstiger Lenovo Thinkcentre server

Vor ein paar Jahren habe ich ein gebrauchtes Lenovo Thinkcentre für nur 80€ gekauft, Baujahr 2016. Seitdem habe ich die interne SSD ausgetauscht, zwei 2 TB HDDs hinzugefügt und den RAM auf 16 GB erweitert. Letzter war vorher in meinem Desktop PC verbaut, bis ich ihn auf 32 GB aufgerüstet habe. Die technischen Daten sind:

- CPU: Intel Core i5-6400
- RAM: 16 GB DDR4
- Interner Speicher: 125 GB SSD für das Betriebsystem
- Massenspeicher: Zwei 2 TB HDDs, eine für Production und eine für Backups
- Betriebssystem: Debian 13

![Kostengünstiger Lenovo Thinkcentre homelab server](/images/posts/homelab-state/pontus.webp?width=300#center "Pontus, mein Server. Der DVD reader fehlt, zum Glück brauche ich ihn nicht.")

Als kleine Office-Workstation braucht das Thinkcentre wirklich wenig Strom. Trotz 2 aktiver HDDs verbraucht er im Idle nur 24 Watt. Energiekosten sind ein wichtiger Faktor für mich, weil Stromkosten in Deutschland ziemlich hoch sind. Trotzdem schalte ich meine Festplatten nicht ab wenn sie nicht gebraucht werden, da ich der Meinung bin, dass es sich durch den höheren Verschleiß finanziell nicht lohnt. Insbesondere bei den aktuellen (August 2026) Preisen.

### Zuverlässige Datenspeicherung mit ZFS unter Linux

ZFS ist der Goldstandard für Dateisysteme. Es hat fortschrittlichen Schutz gegen Datenkorruption, hohe Performance und großartige Unterstützung für Snapshots, welche ich in meinen Backups verwende.  
Für ZFS habe ich mich aus zwei Gründen entschieden: Erstens brauche ich ein zuverlässiges und schnelles Dateisystem für mein Homelab. Ich hoste nicht nur für mich, sondern auch für meine Familie, was ich mich verantwortlich macht, ihre wertvollen Bilder nicht zu verlieren. Zweitens wird ZFS häufig in Unternehmen eingesetzt, sodass ich schon jetzt mit meinem Homelab wichtige Fähigkeiten für potentielle zukünftige Jobs erlernen kann.  

Da die ZFS Lizenz nicht mit dem Linux Kernel kompatibel ist, ist ZFS gewöhnlicherweise nicht standardmäßig in Distributionen enthalten. Das OpenZFS Project entwickelt [OpenZFS on Linux](https://zfsonlinux.org/), welches OpenZFS als so genanntes out-of-tree module mittels DKMS installieren kann. Es wird nur mit stabilen, langsam(er) updatenden Betriebssystemen wie Debian empfohlen, weil ZFS und der Kernel unabhängig voneinander entwickelt werden. Daher sind sie nicht immer aufeinander abgestimmt, und der neuste Kernel muss nicht immer mit OpenZFS kompatibel sein. Außerdem muss ZFS nach der Installation des Betriebssystems installiert werden, was den Aufwand ein neues System zu installieren leicht erhöht. In meinen Augen ist ZFS aber ein sehr guter Kompromiss, und die leicht höhere Arbeit nehme ich gerne in Kauf.

### Docker um meine Homelab Dienste laufen zu lassen

All meine Dienste laufen in [Docker](https://www.docker.com/). Es ist verlässlich und hatte genug Zeit, um seine Qualität unter Beweis zu stellen. Außerdem ist Docker der de-facto Standard für Homelab Anwendungen, sodass fast alle Software die ich hosten möchte als Container Image mit Anleitung für Docker verfügbar ist. Ich benutze docker-compose, um meine Container deklarativ zu verwalten. Dies erlaubt es mir, auf einem neuen System schnell wieder einsatzbereit zu sein.

### Was ich in meinem Homelab hoste

Ein Homelab ist nur so gut, wie die Software die es hostet. Ich habe meine Mission, so viele Big-Tech Dienste wie möglich zu ersetzen mit denen begonnen, die die größte Masse und die wertvollsten Daten haben. Um dabei, auch für meine Familie, die beste Bedienbarkeit zu gewährleisten, habe ich die Voraussetzung, dass alle Dienste Single-Sign-On (SSO) unterstützen. Dank SSO brauchen alle Nutzende sich nur ein mal auf jedem Gerät einloggen, und sind dann sofort für alle Homelab Dienste authentifiziert. Das erspart es ihnen, mit mehreren Nutzernamen und Passwörten hantieren zu müssen.  
Außerdem möchte ich meine Konfiguration so deklarativ wie möglich gestalten. Das bezieht sich auch auf die Konfiguration eines jeden Dienstes. Im Falle eines Systemausfalls möchte ich meine Dienste schnellstmöglich vollumfänglich online bringen können.

#### Authelia für Single Sign On (SSO)

Ich nutze [Authelia](https://www.authelia.com/) um SSO mittels OpenID Connect (OIDC) zu implementieren. Authelia ist eine leichtgewichtige Alternative zu Authentik. Im Gegensatz zu letzterem wird Authelia ausschließlich über deklarative Konfigurationsdateien konfiguriert, inklusive der registrierten Nutzenden. Authelia ist ein zertifizierter OIDC 1.0 Provider, was es kompatibel zu allen Diensten die ebenfalls OIDC 1.0 implementieren macht.

#### Traefik als reverse proxy

Als einziger Server laufen alle Dienste auf Pontus, was einen reverse proxy unabdingbar macht. Es existieren viele gute. Durch meine Arbeit bin ich mit Nginx und Apache httpd vertraut. Für mein Homelab habe ich mich allerdings für Traefik entschieden, weil es nativ mit Docker interagiert, und ich nur ein paar Zeilen pro Dienst schreiben muss, um ihn durch Traefik zugänglich zu machen. Weil die Konfiguration direkt in der `docker-compose.yml` geschieht, muss ich mich außerdem nicht um weitere Konfigurationsdateien kümmern. Traefiks automatische Erneuerung von SSL Zertifikaten ist ebenfalls praktisch. Insbesondere meine Eltern wären verunsicher, wenn ihr Browser auf einmal eine große Warnung anzeigt, wenn sie auf ihre Fotos oder Dokumente zugreifen wollen. 

#### Mesh-VPN mit Tailscale

Mein Homelab soll aus der ganzen Welt erreichbar sein, aber aus Sicherheitsgründen nur für ausgewählte Menschen. [Tailscale](https://tailscale.com/) ist ein moderner Mesh-VPN, welcher Geräte durch Firewalls und NATs miteinander verbinden kann, ohne ein Port in einer Firewall geöffnet werden muss. Tailscale läuft bare-metal auf Pontus. So kann ich Pontus direkt als Tailscale Knoten registrieren, und alle Dienste auf Pontus sind sofort im Tailnet erreichbar. Das nimmt mir die Möglichkeit für Nutzendespezifische Zugangskontrolle, allerdings hat eh nur meine Familie Zugriff und sie können eh alle Dienste nutzen, also brauche ich so etwas momentan nicht.

Leider ist Tailscale ein closed-source, kommerzieller Dienst. Sie haben mindestens Zugriff auf Metadaten, wann welches Gerät sich mit welchen anderen verbindet. In Zukunft würde ich Tailscale gerne durch eine selbst-gehostete Option ersetzen, zum Beispiel [Nebula](https://github.com/slackhq/nebula).

#### Immich als Google Fotos Ersatz

[Immich](https://immich.app/) ist der wichtigste und erste Service in meinem Homelab. Und es ist großartig. Meine Familie und ich lieben es, wir alle speichern unsere Bilder und Videos in Immich. Die eigenen Bilder und Fotos unter der eigenen Kontrolle zu haben ist einer der wichtigsten Schritte um zu de-Googlen, und das macht Immich sehr einfach. Bevor ich Immich hatte, habe ich meine Bilder in unregelmäßigen Abständen auf meinen PC verschoben, und von da aus auf eine externe Festplatte dupliziert. Meine Familie hat einfach Google Fotos benutzt. Dank Immich kann ich alle meine Fotos immer im Browser oder der Handyapp anschauen, während sie sicher auf meinem Server verwahrt sind. Außerdem hat Immich lokale Gesichtserkennung, dank der man schnell nach Bildern mit bestimmten Personen suchen kann. Zusätzlich bietet Immich Objekterkennung, OCR und GPS tags. All das 100% lokal auf meinem alten Server, der dafür überraschend wenig Ressourcen aufwenden muss.  

Ich weiß, dieser Abschnitt liest sich wie ein Werbetext für Immich. Aber Immich ist halt einfach großartig.

#### Paperless-ngx zur Dokumentenverwaltung

Ich hoste auch [Paperless-ngx](https://docs.paperless-ngx.com/) als Dokumentenverwaltung. Ehrlicherweise nutze ich es noch nicht viel, da ich erst kürzlich OIDC in [Paperless mobile](https://play.google.com/store/apps/details?id=de.astubenbord.paperless_mobile), der Android-App, zum Laufen bekommen habe. Sie ermöglicht es, Bilder von Dokumenten als PDFs hochzuladen. Da ich keinen Scanner besitze, sind Bilder die einzige Möglichkeit für mich, analoge Dokumente zu digitalisieren. Wenn ich Zeit habe, muss ich mich mal hinsetzen und einen Haufen digitaler und analoger Dokumente in Paperless einpflegen. 

#### Beszel für Docker-natives Monitoring

[Beszel](https://www.beszel.dev/) ist eine sehr schöne und sehr praktische Anwendung für Monitoring mit eingebautem Support für Docker Container. Beszel selbst wird ebenfalls mit Docker deployed. Zur Zeit gilt meine Hauptsorge der Gesundheit meiner beiden Festplatten. Es ist kein Problem für meine Familie und mich, wenn ein Dienst eine Zeit lang ausfällt, und der Server ist weit entfernt davon überlastet zu sein, sodass wir uns darum auch nicht sorgen müssen. Deutlich problematischer ist der Ausfall einer Festplatte. Ich habe zwar sowohl onsite als auch offside Backups, aber kein Raid, alle Dienste laufen auf einer Festplatte. Daher wüsste ich gerne frühzeitig, wenn eine den Geist aufgibt, sodass ich sie austauschen kann, ohne dass das gesamte Homelab tagelang nicht verfügbar ist, bis eine neue Festplatte geliefert wurde und ich sie eingebaut habe. Docker ermöglicht es, die Festplatten an den Beszel Container weiter zu geben, sodass dieser fortlaufend die SMART Daten aller Festplatten auslesen kann.

![Festplatten SMART Daten Dashboard in Beszel](/images/posts/homelab-state/beszel_smart.png "Festplatten SMART Daten in Beszel")

Beszel ist weniger detailliert und flexibel wie Prometheus + Grafana (welches ich von meiner Arbeit kenne), die beiden wären für mein aktuelles Homelab aber auch übertrieben. Beszel ist eine einfache, schöne Lösung. Genau das, was ich brauche.

#### Datensicherung und Backups

Wie bereits erwähnt, ist die Sicherheit meiner Daten extrem wichtig. Durch mein enges Budget habe ich keinen zweiten onsite Backupserver. Das onsite Backup ist schlicht die zweite 2 TB Festplatte, welche in einem anderen ZFS Pool lebt:

- `data-pool` für Production. Hier sind alle Docker Volumes gespeichert.
- `backup-pool` für Backups. Jede Nacht führt ein Skript einen inkrementellen ZFS Snapshot von `data-pool` aus und sendet ihn zu `backup-pool`.

Sollten während einem Backup Probleme auftreten, werde ich mittels [ntfy](https://ntfy.sh/) informiert.  

Natürlich ist diese Situation nicht ideal. Sie ist nicht fehlertolerant. Falls einer der beiden Festplatten ausfällt, sollte Pontus sofort abgeschaltet werden. Ich *könnte* ihn temporär ohne lokales Backup weiterbetreiben und mich auf mein offsite Backup verlassen, aber ideal wäre das bei weitem nicht.

Mein offsite Backup ist eine 5 TB [Hetzner Storagebox](https://www.hetzner.com/de/storage/storage-box/). Getreu der 3-2-1 Regel werden die Backups hier mit [Borg Backup](https://www.borgbackup.org/) durchgeführt. 

## Media: DIY Satelliten TV

Meine Eltern schauen gerne Satelliten TV, selbst in Zeiten von Netflix und Co. Kommerzielle Receiver sind teuer und voller kommerzieller Software welcher man nur bedingt trauen kann. Insbesondere wenn sie ans Internet angeschlossen werden will, so wie alles heutzutage. Als der Alte meiner Eltern den Geist aufgegeben hat, habe ich einen gebrauchten [Minisforum](https://www.minisforum.com/) Mini-PC und einen TV Stick von [Sundtek](https://www.minisforum.com/) gekauft. Der Mini-PC, welchen ich schlicht *media* nenne, ist an ihren Fernseher über HDMI angeschlossen. Zum Glück ist der Fernseher zu alt, um ein internet-abhängiger Smart-TV zu sein.  

Das Betriebssystem von *media* ist ebenfalls Debian 13. Überall das gleiche Betriebssystem zu verwenden, verringert den Wartungsaufwand. Nach einem Start loggt der PC sich automatisch in einen Account ohne sudo und root Rechte ein und startet [Kodi](https://kodi.tv/), welches meine Eltern mit [Kore](https://play.google.com/store/apps/details?id=org.xbmc.kore) übers Smartphone vom Sofa aus steuern können. Falls meine Eltern einen Streaming Service nutzen wollen, können sie die dauerhaft angeschlossene Tastatur und Maus verwenden.  

Der Sundtek TV Stick ist via USB angeschlossen und bekommt sein DVB-S signal von der Satellitenschüssel. Sundtek bietet aktuelle Treiber für Linux, welche auch einen Systemd service installieren, der den Stick automatisch nach dem Bootvorgang startet.

## Offsite Backups für den Ernstfall

Als Offsite Backups nutze ich eine [Hetzner Storagebox](https://www.hetzner.com/de/storage/storage-box/) mit 5 TB mit [Borg Backup](https://www.borgbackup.org/). Nicht nur *pontus* sichert seine Daten dorthin, sondern auch die Laptops meiner Eltern, sowie mein Laptop und PC. Die 5 TB sollten dafür locker ausreichen. Jedes Gerät hat einen eigenen Subaccount mit eigenem SSH Schlüssel auf der Storage Box und nur Zugriff auf sein Borg Repository. Die Repositories sind selbstverständlich verschlüsselt, inkrementelle Backups geschehen täglich.  

Da Borg eine andere Technologie als ZFS ist, erfüllt das System auch die "1" der 3-2-1 Regel, welche besagt, dass mindestens ein Backup mit einer anderen Technologie gemacht werden sollen.