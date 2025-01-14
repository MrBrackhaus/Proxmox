---
title: "Einrichtung und Verwaltung einer sicheren Proxmox VE Testumgebung"
author: "Michael Kurz, Fachinformatiker – Systemintegration"
date: "2025-01-14"
version: "2.1"
description: "Ein umfassender Leitfaden zur Einrichtung und Verwaltung einer sicheren und skalierbaren Proxmox VE Testumgebung mit Best Practices für Storage, Netzwerke und Hochverfügbarkeit."
categories:
  - Virtualisierung
  - Proxmox VE
  - IT-Sicherheit
  - Netzwerkkonfiguration
  - Open Source
tags:
  - Proxmox
  - Virtualisierung
  - Storage
  - Clustering
  - Hochverfügbarkeit
  - Sicherheit
  - ZFS
  - Netzwerkdesign
license: "GPL-3.0"
toc: true
keywords:
  - Proxmox Virtual Environment
  - Testumgebung
  - ZFS
  - Cluster-Setup
  - Sicherheit
  - VLAN
  - Backup
language: "de"
output:
  html_document:
    toc: true
    toc_depth: 3
    number_sections: false
    code_folding: hide

---

# Einrichtung und Verwaltung einer sicheren Proxmox VE Testumgebung

# Vorwort**

In der modernen IT-Landschaft sind Anforderungen wie **Agilität**, **Skalierbarkeit** und **Sicherheit** nicht mehr nur „nice to have“ – sie sind essenzielle Bausteine, um wettbewerbsfähig zu bleiben. Daten müssen schnell bereitgestellt, Services on demand erstellt und bei Bedarf wieder abgebaut werden können. Gleichzeitig steigen die Erwartungen hinsichtlich Performance und Ausfallsicherheit. Diese Trends zusammen haben der **Virtualisierung** einen enormen Schub verpasst.

**Proxmox Virtual Environment (Proxmox VE)** hat sich hier als eine der führenden Open-Source-Lösungen etabliert, die sowohl die **Vollvirtualisierung** (KVM) als auch die **Container-Virtualisierung** (LXC) unter einem Dach vereint. Das bedeutet:

* Sie können **klassische virtuelle Maschinen** betreiben, in denen komplette Betriebssysteme laufen.
* Sie können **leichtere Container** nutzen, um einzelne Dienste isoliert zu fahren, was Ressourcen schont und Deployments beschleunigt.

Dieses Fachbuch führt Sie Schritt für Schritt durch die Konzeption und den Aufbau einer **Testumgebung** auf Basis von Proxmox VE. Dabei legen wir hohen Wert auf **Sicherheit** und **Best Practices**, damit Sie nicht nur schnell, sondern auch **solide** eine leistungsfähige Umgebung auf die Beine stellen.

## Zielgruppe

* **Fachinformatiker\*innen für Systemintegration:**\
  Menschen mit grundlegenden IT-Infrastruktur-Kenntnissen, die sich tiefer in Virtualisierung und Cloud-nahe Themen einarbeiten wollen.
* **IT-Professionals und Administrator\*innen:**\
  Bereits erfahren mit Linux, wollen eine **weitere** Virtualisierungsplattform evaluieren oder Proxmox VE in einem Testlabor aufsetzen.
* **Technische Enthusiast\*innen:**\
  Möchten in ihrer Freizeit oder als Hobby eine leistungsstarke, kostengünstige (da open source) Virtualisierungsumgebung bauen.

Dabei ist klar: Der Sprung von einer reinen „Testumgebung“ hin zu **produktionsnahen Setups** (hochverfügbare Cluster, verschiedene Standorte etc.) ist fließend – Proxmox VE eignet sich durchaus für beides.

## Gründe für Proxmox VE

1. **Vollständige Integration von KVM und LXC**

   * Sie können VMs (Windows, Linux, BSD etc.) nutzen oder Linux Container hochfahren – alles in einer Management-Oberfläche.

2. **Einfache Bedienung via WebGUI**

   * Zentrales Management, konsistente Menüführung.

3. **Leistungsstark und flexibel**

   * **ZFS**-Integration, **Ceph**-Support, verteilte Cluster-Features.
   * Integrierte Backup-Funktionen, HA (Hochverfügbarkeit), Live Migration, usw.

4. **Offene Architektur**

   * Auf Debian/Linux basierend, daher gut konfigurierbar, Skripting-freundlich, großer Community-Support.

5. **Umfassende Sicherheitskonzepte**

   * Proxmox-Firewall, Container-Isolation, **RBAC**-Zugriffsmodell, 2FA.
   * Integration von Tools wie CrowdSec, Fail2Ban, IDS/IPS-Systemen.

Gerade in einer **Testumgebung**, in der neue Dienste oder Betriebssysteme erprobt werden sollen, ist Proxmox VE dank seiner übersichtlichen GUI und den integrierten Snapshot-/Backup-Funktionen sehr angenehm zu administrieren.

## Aufbau und Umfang dieses Fachbuchs

Wir gehen **deutlich über eine Kurzanleitung** hinaus, um Ihnen die wichtigsten Hintergrundinformationen zu liefern. Nach der Lektüre sollten Sie in der Lage sein:

* Eine **saubere Installation** von Proxmox VE vorzunehmen (inklusive Feinheiten wie Partitionierungsoptionen, ZFS vs. EXT4).
* **Storage** (ZFS, BTRFS, EXT4) vernünftig zu konfigurieren, Snapshots zu nutzen und Performance zu optimieren.
* **Netzwerk**-Belange (Bridging, Bonding, VLANs) abzusichern und mit Firewalls/Segmentierung zu arbeiten.
* **Sicherheitsmaßnahmen** (SSH-Härtung, 2FA, Keycloak als SSO, CrowdSec) umfassend einzusetzen.
* **Cluster- und HA-Funktionen** einzurichten, inklusive Live Migration und (optional) Georedundanz.
* **Backup-, Monitoring- und Automatisierungsstrategien** umzusetzen, die langfristig Stabilität und Übersicht schaffen.

Jedes Kapitel baut dabei **nicht zwingend** auf das vorherige auf, kann aber in einer logischen Reihenfolge gelesen werden (z.B. zuerst Installation, dann Storage, dann Netzwerk, dann Sicherheit etc.).

***

## Danksagung**

Ein solches Werk entspringt meist dem **Engagement** und der **Expertise vieler Personen**, sei es in Forenbeiträgen, Blog-Artikeln oder direktem Austausch. Deshalb gebührt mein Dank:

1. **Open-Source-Community**\
   Jeder, der/die in Foren, auf GitHub oder in Mailinglisten Beiträge leistet, Code testet oder an Proxmox VE und dessen Ökosystemen (z.B. ZFS on Linux, LXC, Debian) arbeitet. Ohne diese Community wäre eine so umfangreiche Software wie Proxmox kaum möglich.

2. **Kolleginnen und Kollegen**\
   Die zahlreichen Diskussionen um Netzwerk-Setups, Cluster-Desaster und Storage-Tuning haben direkt in diese Dokumentation Einzug gefunden. Viele der Tipps und Tricks kommen aus dem Alltag von Menschen, die Proxmox VE produktiv betreiben.

3. **Mentoren und IT-Professionals**\
   Ob durch Schulungen, Online-Kurse oder praktische Erfahrungen – Mentoren haben meine Sicht auf Virtualisierung und IT-Architekturen geprägt. Sie haben den Blick für das „große Ganze“ geschärft: **Technik** ist nur ein Teil; **Prozesse, Wartbarkeit** und **Sicherheit** sind ebenso wichtig.

4. **Nutzerfeedback**\
   Jede Rückmeldung, jede Frage („Warum geht das so nicht?“), hat den Inhalt feingeschliffen. Besondere Danksagung an all jene, die kritische Fehler entdeckten oder tiefergehende Nachfragen stellten – nur so kann eine Anleitung „rund“ werden.

***

## Hinweise zur Nutzung dieses Fachbuchs

* **Markdown-Form:** Dieser Text liegt in Markdown vor. Kopieren Sie ihn in Zettlr, VSCode oder einen anderen Markdown-Editor, um bequem zu navigieren, Notizen einzufügen und Abschnitte auszublenden.
* **Scrollbare Codeblöcke, Beispiel-Befehle:** Achten Sie auf die Hervorhebungen (`**fett**`, `*kursiv*`, `bash code`), die Ihnen helfen, Befehle direkt zu übernehmen.

> **Tipp**: Führen Sie parallel eine **Testinstallation** durch. Lesen Sie das jeweilige Kapitel, setzen Sie die Schritte um und notieren Sie eigene Erkenntnisse in den Kommentaren oder internen Dokumenten.

***

## Warum wir uns besonders auf Sicherheit und Best Practices fokussieren

1. **Virtualisierung als kritische Infrastruktur**

   * Ein Proxmox VE Host, der mehrere produktionsrelevante Server trägt, wird schnell zum Single Point of Failure. Wenn er kompromittiert wird, kann das ganze Unternehmen zum Stillstand kommen.

2. **Testumgebung kann auf Produktivniveau wachsen**

   * Oft fängt man mit ein paar VMs an („nur Testumgebung“) und merkt, wie stabil und bequem Proxmox VE ist. Dann zieht man mehr Services rüber, plötzlich wird’s produktionsähnlich.

3. **Best Practices**

   * Mit sauberer Trennung (Storage, Netzwerk, Sicherheitslayer) legen Sie heute schon den Grundstein, dass Ihre Proxmox-Installation **erweiterbar** bleibt: Egal ob Sie 1, 2 oder 10 Knoten später haben, Sie stehen nicht vor fundamentalen Redesigns.

4. **Konfliktvermeidung**

   * Viele Anleitungen unterschätzen, dass man in einer aktiven IT-Umgebung mit existierenden Switch-Konfigurationen, Firewalls, IP-Adressplänen und Richtlinien interagiert. Unsere Dokumentation zeigt, wie man Proxmox VE so integriert, dass es **möglichst wenige Reibungspunkte** gibt.

***

# **Kapitel 1 – Voraussetzungen**

In vielen Anleitungen wird der Bereich „Voraussetzungen“ nur kurz abgehandelt – jedoch liegt gerade hier der Schlüssel zum Erfolg. Wenn bereits **Hardware** und **Netzwerk** richtig durchdacht sind, spart man sich später viel Frust. Ebenso spielt die **Planung** (z.B. Speicherplatz, Ausfallsicherheit, Updates) eine wichtige Rolle. Dieses Kapitel ist deshalb umfangreicher, um Ihnen alle nötigen Hintergrundinfos zu geben.

## 1.1 Hardware-Anforderungen

### 1.1.1 Prozessor (CPU)

* **Empfohlen**: x86\_64-Prozessor mit aktivierten Virtualisierungsfunktionen wie Intel VT-x oder AMD-V.

* **Warum so wichtig?**

  1. **KVM** (Kernel-based Virtual Machine) nutzt Hardware-beschleunigte Virtualisierung. Ohne diese CPU-Erweiterungen läuft KVM im sog. Software-Fallback-Modus, was **merkliche Performanceeinbußen** bringt.
  2. Bei Containern (LXC) ist die CPU-Anforderung zwar geringer, aber sofern man Windows-VMs oder andere komplexe Betriebssysteme virtualisieren möchte, profitiert man stark von einer hardware-basierten Beschleunigung.

* **Praxistipp**:

  * Intel Xeon E5/E3, neuere AMD EPYC oder Ryzen-Systeme funktionieren in der Regel sehr gut.
  * Achten Sie auch auf den **Stromverbrauch**; in einer Testumgebung kann ein energieeffizienter Prozessor genügen, sofern man keine Hochleistungsserver simulieren muss.

### 1.1.2 Arbeitsspeicher (RAM)

* **Minimum**: 16 GB RAM

* **Empfehlung**: 32 GB oder mehr

* **Hintergrund**:

  1. Bei intensiver Virtualisierung läuft schnell eine Reihe von VMs parallel. Jede VM benötigt einen eigenen RAM-Anteil.
  2. **ZFS** (falls verwendet) hat eine Faustregel: 1 GB RAM pro Terabyte verwalteter Storage-Kapazität. Werden also 4 TB Kapazität im RAID-Z eingebunden, sollte man mind. 4-8 GB RAM allein für ZFS einplanen.
  3. Für Container (LXC) ist der RAM-Verbrauch zwar schlanker als bei VMs, aber man sollte immer Reserven haben, um spontane Tests oder Snapshots durchführen zu können.

* **Profi-Hinweis**:

  * OD/ECC-RAM (On-Die/Error-Correcting Code) kann Defekte frühzeitig entdecken. Gerade bei ZFS oder produktionsnahen Setups empfehlenswert, weil Speicherfehler sonst die Datenintegrität beeinträchtigen könnten.

### 1.1.3 Speicher (Disks)

* **Systemlaufwerk**: Idealerweise eine SSD oder NVMe für das Proxmox-Betriebssystem.

* **Storage-Laufwerke**:

  1. **ZFS** (z.B. RAID-Z oder Mirror) zur besseren Datenintegrität, Snapshots, Kompression.
  2. **Alternative**: EXT4 (klassisch, stabil, weniger Features) oder BTRFS (Snapshots, Subvolumes, teils experimentelle RAID5/6).
  3. Wer **Ceph** in einem Cluster plant, könnte später OSDs auf separaten Festplatten einrichten.

* **Wichtige Überlegung**:

  * Möchten Sie in einer Testumgebung viele große VMs fahren (z.B. Windows-VMs mit jeweils 100+ GB Disk)? Dann sollten Sie an ausreichend Kapazität und ggf. **Redundanz** denken.
  * Lese-/Schreib-Performance ist bei Virtualisierung ausschlaggebend für ein „flüssiges“ Gefühl in den VMs.

### 1.1.4 Netzwerkadapter

* **Mindestens**: 1 Gbit/s pro Node – realistisch aber mehr, wenn Cluster und Storage-Verkehr zusammenlaufen.

* **Empfehlung**: 2 oder mehr Ports für **Bonding** (Ausfallsicherheit / LACP).

* **Warum?**

  1. Knoten-Kommunikation (Corosync, HA Heartbeat, Live Migration) erfordert stabilen, **verlustarmen** Durchsatz.
  2. In produktionsnahen Test-Szenarien wird häufig VLAN-Trennung (Management, Storage, DMZ) angewendet. Mehr physische Ports oder trunk-fähige Switches erleichtern das Setup.

### 1.1.5 USV (Optionale Empfehlung)

* Für einen Testlabors ist es kein Muss, dennoch kann eine **USV** (Unterbrechungsfreie Stromversorgung) plötzliche Spannungsausfälle abfangen. Gerade **ZFS** reagiert empfindlich auf harte Stromausfälle, da Schreibvorgänge unvollständig sein könnten.
* Wer z.B. Hochverfügbarkeit testet, sollte zudem einen abgesicherten Switch haben, sonst bleibt der Heartbeat bei Stromausfall stehen.

***

## 1.2 Software-Anforderungen

### 1.2.1 Proxmox VE ISO

* **Download**: Von [Proxmox.com](https://www.proxmox.com/en/downloads), aktuellste Version (z.B. Proxmox VE 7.x oder neuer).
* **Checksum**: Prüfen Sie den SHA256-Hash, damit keine korrupte ISO verwendet wird.

### 1.2.2 Debian-Kenntnisse

* Proxmox VE basiert auf **Debian**. Ein **Basiswissen** in apt-Paketverwaltung, Netzwerk-CLI (ip/ifconfig), systemd-Diensten kann enorm helfen.
* Auch ein generelles Verständnis von Linux-Dateisystemstruktur (/etc, /var/log, etc.) erspart viele Kopfschmerzen.

### 1.2.3 Bootfähiger USB-Stick

* Mindestens 8 GB Kapazität.
* Tools: **Rufus** (Windows) oder **balenaEtcher** (Linux/macOS).
* Achten Sie auf **UEFI** vs. Legacy-BIOS-Einstellungen (manche USB-Sticks booten nur, wenn Secure Boot ausgeschaltet ist).

### 1.2.4 Enterprise vs. Community Repository

* **Enterprise-Repo**: Stabilere, getestete Updates, benötigt Proxmox-Subscription (kostenpflichtig).
* **Community-Repo**: Kostenfrei, sehr aktuell, jedoch potenziell weniger gründlich getestete Paketstände. Für Testumgebungen typischerweise ausreichend.

> **Extratipp**: Gerade anfangs kann man das Community-Repo nutzen. Sollte die Umgebung später produktiv werden, kann man eine Subscription erwerben und auf Enterprise-Repos umstellen.

### 1.2.5 Zeit- und DNS-Dienste

* **NTP**: Ungenaue Systemzeiten führen schnell zu Token-/SSL-Fehlern oder Cluster-Desynchronisierung.
* **DNS**: Korrekte Forward- und Reverse-Lookups (pve01.example.local → 192.168.1.x, etc.) sind essenziell für Clustereinrichtung, Live Migration und HA.

***

## 1.3 Netzwerk-Anforderungen

Ein gut durchdachtes Netzwerkdesign ist von unschätzbarem Wert. So vermeiden Sie, dass Knoten sich nicht finden oder Heartbeat/Corosync-Links flappen.

### 1.3.1 Lokale Infrastruktur & Switches

* **VLAN-Unterstützung**: Switches sollten VLAN-Tagging (IEEE 802.1Q) sauber beherrschen, falls Sie Subnetze trennen.
* **LACP**: Wenn Sie Bonding (802.3ad) planen, muss der Switch LACP-Ports konfigurieren können.

### 1.3.2 IP-Adressierung

* Statisch vs. DHCP:

  * _Statisch_: Empfohlen für Management-Schnittstellen, um Ausfälle oder IP-Änderungen zu vermeiden.
  * _DHCP_: Möchte man VMs flexibel hochfahren, kann man DHCP in den Gastsystemen nutzen, jedoch nicht für die Proxmox-Hosts selbst.

### 1.3.3 Firewall / Router

* Legen Sie fest, ob Sie eine **Hardware-Firewall** (z.B. pfSense-Box) nutzen oder ob Proxmox-Firewall ausreicht.
* Wenn Sie in der Testumgebung auch Internet-Anbindung benötigen, sollten Sie klären, wer NAT, Portforwarding und VPN-Zugänge verwaltet.

### 1.3.4 Öffentlicher Zugriff (Optional)

* Für reine Testumgebungen häufig gar nicht notwendig.
* Falls gewünscht: Achten Sie auf SSL-Zertifikate und Firewalls. Port 8006 (Proxmox-GUI) sollte nur über ein sicheres VPN oder eine dedizierte Firewall-Freigabe erreichbar sein.

***

## 1.4 Fazit „Voraussetzungen“

Wer eine **saubere** Proxmox-Testumgebung haben will, kann bereits hier viel falsch machen, indem man z.B.

* Beliebige Hardware zusammenwirft ohne RAID/Redundanz oder
* CPU ohne VT-Erweiterungen oder
* wackelige Netzwerkkonfiguration\
  … und sich dann wundert, warum Clustering oder Snapshots nicht zuverlässig sind.

**Checkliste**:

1. **CPU** mit VT-x/AMD-V, ausreichend Cores je nach VM-Anzahl.
2. **RAM**: 16 GB Minimum, 32 GB+ empfohlen (ZFS = mehr RAM).
3. **Storage**: SSD/NVMe oder HDD + ZFS? RAID-Spiegel, ECC-RAM?
4. **Netzwerk**: Mind. 1 Gbit/s, besser Bonding, VLAN-tauglicher Switch.
5. **Software**: Aktuelles Proxmox VE ISO, Debian-Basiswissen, Repos (Community vs. Enterprise).
6. **NTP, DNS**: Korrekte Zeitsynchronisierung, sauberer Hostname, /etc/hosts-Einträge gepflegt.

***

# **Kapitel 2 – Installation von Proxmox VE**

Nachdem wir nun genau wissen, wie unsere Hardware/Software-Ausstattung aussehen soll, schreiten wir zur **Installation**. Dabei behandeln wir sowohl die ISO-basierte Vorgehensweise als auch ein paar Hintergründe (Partitionierungsoptionen, Post-Install-Schritte).

## 2.1 ISO-Vorbereitung und Boot-Vorgang

### 2.1.1 ISO-Download

1. **Webseite**: [proxmox.com/en/downloads](https://www.proxmox.com/en/downloads)

2. **Datei**: `proxmox-ve_*.iso` – je nach Version (z.B. 7.x, 8.x).

3. **Hash-Prüfung**:

   * Öffnen Sie eine Konsole, führen Sie `sha256sum proxmox-ve_*.iso` aus und vergleichen Sie den Wert mit dem auf der Proxmox-Download-Seite angegebenen.
   * Bei Abweichungen ISO neu herunterladen.

***

### **2.1.2 Erstellen eines bootfähigen USB-Sticks**

Ein bootfähiger USB-Stick wird benötigt, um die Proxmox VE ISO auf dem Zielsystem zu installieren. Im Folgenden werden die Schritte für Windows, Linux und macOS beschrieben. Zudem werden wichtige Punkte zur Auswahl von UEFI oder Legacy-Boot erläutert.

***

#### **Vorbereitung:**

1. **Benötigte Materialien:**

   * Einen USB-Stick mit mindestens **8 GB Speicherplatz**.
   * Die Proxmox VE ISO-Datei, die von der [offiziellen Proxmox-Website](https://www.proxmox.com/en/downloads) heruntergeladen wurde.

2. **Wichtig:**

   * **Sichern Sie alle Daten auf dem USB-Stick**, da dieser während der Erstellung formatiert wird.
   * Vergewissern Sie sich, dass Ihr System das richtige Bootverfahren unterstützt (UEFI oder Legacy, siehe unten).

***

#### **Windows: Erstellung mit Rufus**

**Rufus** ist ein weit verbreitetes Tool zur Erstellung bootfähiger USB-Sticks auf Windows-Systemen.

1. **Rufus herunterladen und starten:**

   * [Rufus-Website](https://rufus.ie/) besuchen, die neueste Version herunterladen und ausführen (keine Installation notwendig).

2. **USB-Stick auswählen:**

   * Schließen Sie den USB-Stick an und wählen Sie ihn im Dropdown-Menü unter „Gerät“ aus.

3. **ISO-Datei auswählen:**

   * Klicken Sie auf „Auswählen“ und navigieren Sie zur heruntergeladenen Proxmox VE ISO.

4. **Partitionierungsschema überprüfen:**

   * Wählen Sie das passende Partitionierungsschema:

     * **MBR:** Für ältere Systeme mit BIOS oder Legacy-Boot.
     * **GPT:** Für moderne Systeme mit UEFI-Boot.

   * Rufus schlägt basierend auf der ISO automatisch ein Schema vor; ändern Sie dies nur, wenn Sie sicher sind.

5. **Dateisystem und Optionen:**

   * Dateisystem: **FAT32** (Standard).
   * Häkchen bei „Schnellformatierung“ setzen.

6. **Erstellung starten:**

   * Klicken Sie auf „Start“.
   * Rufus zeigt einen Warnhinweis, dass alle Daten auf dem Stick gelöscht werden. Bestätigen Sie mit „OK“.

7. **Abschluss:**

   * Sobald Rufus meldet, dass der Vorgang abgeschlossen ist, können Sie den USB-Stick sicher entfernen.

***

#### **Linux/macOS: Erstellung mit Etcher**

**Etcher** ist ein einfach zu bedienendes Tool, das unter Linux und macOS gleichermaßen gut funktioniert.

1. **Etcher herunterladen:**

   * Besuchen Sie die [Etcher-Website](https://etcher.balena.io/) und laden Sie die passende Version für Ihr Betriebssystem herunter.

2. **Etcher starten:**

   * Starten Sie Etcher nach der Installation.

3. **ISO-Datei auswählen:**

   * Klicken Sie auf „Flash from file“ und wählen Sie die Proxmox VE ISO-Datei aus.

4. **USB-Stick auswählen:**

   * Schließen Sie den USB-Stick an und klicken Sie auf „Select target“. Wählen Sie den USB-Stick aus der Liste.

5. **Schreibvorgang starten:**

   * Klicken Sie auf „Flash“ und bestätigen Sie ggf. die Administratorrechte.
   * Hinweis: Etcher löscht alle Daten auf dem Stick.

6. **Abschluss:**

   * Sobald der Vorgang abgeschlossen ist, erhalten Sie eine Bestätigung. Entfernen Sie den USB-Stick sicher.

***

#### **Manuelle Erstellung (Linux/Terminal)**

Falls Sie kein Tool wie Etcher verwenden möchten, können Sie den USB-Stick auch direkt über das Terminal erstellen.

1. **ISO-Datei ermitteln:**

   * Wechseln Sie in das Verzeichnis, in dem sich die ISO befindet (z. B. `cd ~/Downloads`).

2. **USB-Stick identifizieren:**

   * Schließen Sie den USB-Stick an und prüfen Sie, wie er im System erkannt wird:
     ```bash
     lsblk
     ```
     Notieren Sie den Gerätenamen (z. B. `/dev/sdb`).

3. **ISO auf USB-Stick schreiben:**

   * Verwenden Sie den folgenden Befehl:
     ```bash
     sudo dd if=proxmox-ve.iso of=/dev/sdX bs=4M status=progress && sync
     ```
     Ersetzen Sie `proxmox-ve.iso` durch den Dateinamen der ISO und `/dev/sdX` durch den Gerätenamen Ihres USB-Sticks.
   * **Achtung:** Stellen Sie sicher, dass Sie den richtigen Gerätenamen verwenden, um keine anderen Laufwerke zu überschreiben.

4. **Abschluss:**

   * Warten Sie, bis der Vorgang abgeschlossen ist, und entfernen Sie den USB-Stick sicher.

***

#### **UEFI vs. Legacy Boot**

**UEFI und Legacy Boot** sind die beiden häufigsten Bootverfahren. Moderne Systeme unterstützen in der Regel UEFI, während ältere Hardware auf Legacy-Boot angewiesen ist.

1. **UEFI:**

   * Vorteile:

     * Schnellere Bootzeit.
     * Unterstützung für größere Partitionen und moderne Sicherheitsfunktionen (z. B. Secure Boot).

   * Konfiguration:

     * Aktivieren Sie im BIOS den **UEFI-Mode**.
     * **Secure Boot:** Proxmox unterstützt Secure Boot nicht immer nativ. Deaktivieren Sie diesen Modus, falls der Bootvorgang fehlschlägt.

2. **Legacy Boot:**

   * Vorteile:
     * Kompatibilität mit älteren Systemen.
   * Konfiguration:
     * Wechseln Sie im BIOS in den **Legacy-Mode** oder aktivieren Sie **CSM (Compatibility Support Module)**.

3. **BIOS/UEFI aufrufen:**

   * Beim Start des Computers die entsprechende Taste drücken (oft **F2**, **DEL**, **ESC** oder **F10**, je nach Hersteller).
   * Stellen Sie sicher, dass der USB-Stick als primäres Bootmedium eingerichtet ist.

***

#### **Tipps und häufige Probleme:**

* **USB-Stick wird nicht erkannt:**
  * Stellen Sie sicher, dass der Stick korrekt formatiert wurde und im BIOS als Bootmedium erkannt wird.
* **Fehlermeldungen bei Rufus oder Etcher:**
  * Verwenden Sie einen anderen USB-Stick oder Port.
* **Probleme beim Booten:**
  * Prüfen Sie die BIOS/UEFI-Einstellungen. Achten Sie insbesondere auf den Boot-Modus (UEFI/Legacy).

Mit einem korrekt erstellten USB-Stick und den richtigen BIOS-Einstellungen sind Sie bereit, die Installation von Proxmox VE zu starten!


### 2.1.3 Systemstart vom Stick

1. **Rechner einschalten**, Taste für Bootmenü (F12, F2, DEL – je nach Hersteller) drücken.
2. **USB-Laufwerk** als Bootmedium auswählen.
3. Proxmox-Bootmenü erscheint, hier: „**Install Proxmox VE**“ wählen.

> **Hinweis**: Falls Ihr System nicht vom USB bootet, prüfen Sie Bootreihenfolge und Secure-Boot-Einstellungen.

***

### **2.1.4 Systemstart von PXE (Preboot Execution Environment)**

Das PXE-Boot-Verfahren ermöglicht es, ein Betriebssystem oder ein Installationsprogramm wie Proxmox VE direkt über das Netzwerk zu laden, ohne ein physisches Medium (z. B. USB-Stick oder CD) zu benötigen. Diese Methode ist besonders nützlich in größeren IT-Umgebungen oder bei Headless-Servern.

***

#### **Was ist PXE?**

**PXE (Preboot Execution Environment)** ist ein Netzwerkprotokoll, das es Computern erlaubt, ein Betriebssystem-Image oder Installationsdateien direkt von einem Server zu laden. Der Boot-Prozess erfolgt in mehreren Schritten:

1. **DHCP-Anfrage:** Der Client (das Zielsystem) fordert eine IP-Adresse und PXE-Informationen vom Netzwerk-DHCP-Server an.
2. **TFTP-Bootloader:** Der PXE-Server liefert einen Bootloader (z. B. iPXE oder GRUB), der die Steuerung übernimmt.
3. **Kernel und Initramfs:** Der Bootloader lädt die benötigten Dateien (Kernel, Initramfs) über TFTP oder HTTP.
4. **Start des Installationsprogramms:** Die Installationsroutine wird gestartet, z. B. die Proxmox VE Installation.

***

#### **Voraussetzungen für PXE-Boot**

1. **Netzwerk-Infrastruktur:**

   * Ein DHCP-Server im Netzwerk, der PXE-Boot-Optionen unterstützt.
   * Ein PXE-Server mit TFTP- und ggf. HTTP/FTP-Unterstützung.

2. **Proxmox VE ISO-Dateien:**

   * Die ISO-Datei muss auf dem PXE-Server bereitgestellt werden.
   * Extrahieren Sie die Kernel- und Initramfs-Dateien aus der ISO (Anleitung siehe unten).

3. **Client-System:**

   * Unterstützt PXE-Boot im BIOS/UEFI (kann in den Boot-Einstellungen aktiviert werden).

***

#### **PXE-Server einrichten**

Hier ist eine Anleitung zur Einrichtung eines PXE-Servers unter Linux (Debian/Ubuntu).

1. **TFTP-Server installieren:**

   * Installieren Sie einen TFTP-Server:
     ```bash
     sudo apt update
     sudo apt install tftpd-hpa
     ```

   * Konfigurieren Sie den Server:

     ```bash
     sudo nano /etc/default/tftpd-hpa
     ```

     Stellen Sie sicher, dass die Konfiguration wie folgt aussieht:

     ```
     TFTP_OPTIONS="--secure"
     TFTP_DIRECTORY="/srv/tftp"
     RUN_DAEMON="yes"
     OPTIONS="--secure"
     ```

   * Erstellen Sie das Verzeichnis für die TFTP-Dateien und starten Sie den Dienst:
     ```bash
     sudo mkdir -p /srv/tftp
     sudo systemctl restart tftpd-hpa
     ```

2. **DHCP-Server konfigurieren:**

   * Bearbeiten Sie die DHCP-Konfiguration (z. B. für `isc-dhcp-server`):

     ```bash
     sudo nano /etc/dhcp/dhcpd.conf
     ```

     Fügen Sie folgende Optionen hinzu:

     ```
     next-server <IP-Adresse-des-PXE-Servers>;
     filename "pxelinux.0";
     ```

   * Starten Sie den DHCP-Server neu:
     ```bash
     sudo systemctl restart isc-dhcp-server
     ```

3. **PXE-Bootloader bereitstellen:**

   * Installieren Sie `syslinux` für den PXE-Bootloader:
     ```bash
     sudo apt install syslinux
     ```
   * Kopieren Sie die Bootloader-Datei in das TFTP-Verzeichnis:
     ```bash
     sudo cp /usr/lib/PXELINUX/pxelinux.0 /srv/tftp/
     sudo cp -r /usr/lib/syslinux/modules/bios/* /srv/tftp/
     ```

4. **Proxmox VE Kernel und Initramfs bereitstellen:**

   * Mounten Sie die Proxmox VE ISO:
     ```bash
     sudo mount -o loop proxmox-ve.iso /mnt
     ```
   * Kopieren Sie die Dateien `vmlinuz` und `initrd.img` aus dem Verzeichnis `/mnt/boot` in das TFTP-Verzeichnis:
     ```bash
     sudo cp /mnt/boot/vmlinuz /srv/tftp/
     sudo cp /mnt/boot/initrd.img /srv/tftp/
     ```

5. **Boot-Konfigurationsdatei erstellen:**

   * Erstellen Sie eine PXE-Boot-Konfigurationsdatei:

     ```bash
     sudo nano /srv/tftp/pxelinux.cfg/default
     ```

     Beispielkonfiguration:

     ```
     DEFAULT proxmox
     LABEL proxmox
         KERNEL vmlinuz
         APPEND initrd=initrd.img root=/dev/ram0 rw quiet
     ```

***

#### **PXE-Boot am Client aktivieren**

1. **BIOS/UEFI-Einstellungen:**

   * Rufen Sie das BIOS/UEFI des Zielsystems auf (z. B. durch Drücken von **F2**, **DEL**, oder **F12** beim Start).
   * Aktivieren Sie die Option **PXE Boot** oder **Network Boot**.
   * Stellen Sie sicher, dass das Netzwerkgerät als erstes Bootmedium priorisiert ist.

2. **Netzwerkverbindung:**

   * Verbinden Sie das System per Ethernet-Kabel mit dem Netzwerk, in dem der PXE-Server aktiv ist.

3. **PXE-Boot starten:**

   * Beim Start des Systems wird automatisch der PXE-Bootprozess initiiert, sofern korrekt konfiguriert.

***

#### **Tipps und Fehlerbehebung**

* **PXE-Server nicht erreichbar:**

  * Stellen Sie sicher, dass der PXE-Server und der Client im selben Subnetz sind.
  * Prüfen Sie, ob TFTP- und DHCP-Dienste laufen (`sudo systemctl status tftpd-hpa`).

* **Fehlende Kernel- oder Initramfs-Dateien:**

  * Vergewissern Sie sich, dass die Dateien korrekt kopiert wurden.
  * Überprüfen Sie die Pfade in der PXE-Konfigurationsdatei.

* **Langsamer Bootprozess:**

  * Verwenden Sie HTTP/FTP statt TFTP, um größere Dateien schneller zu laden.

***

Mit dieser PXE-Einrichtung können Sie Proxmox VE komfortabel und effizient über das Netzwerk auf Ihren Systemen bereitstellen – ideal für Szenarien mit mehreren Systemen oder ohne direkten physischen Zugriff auf die Geräte.


## 2.2 Installation – Schritt für Schritt

Die meisten Schritte sind relativ intuitiv, dennoch lohnt sich ein genauer Blick auf Partitionierung und Netzwerkkonfiguration, um spätere Migrationen zu ersparen.

1. **Lizenzvereinbarung**
![46a0651de75517bfe12d180173971214](https://github.com/user-attachments/assets/7dd241b5-0e64-49fb-be7d-8c569482ce01)

   * Proxmox VE steht unter AGPL v3. Kurze Zusammenfassung: Der Quellcode bleibt frei zugänglich, aber man akzeptiert die Lizenzbedingungen.

2. **Ziel-Festplatte**
![962afd9c824d88206a31bde0e433ab72](https://github.com/user-attachments/assets/b4d88dd3-1669-48d6-a81a-4017d2b1e5ea)
   * Im Installer können Sie eine (oder mehrere) Festplatten wählen.
   * Falls Sie **ZFS** als Root-FS bevorzugen, wählen Sie „**ZFS RAID**“.
     * Wählen Sie das RAID-Level (z.B. RAIDZ1 bei 3 Platten) oder Mirror (2 Platten).
       ![833cc65b91df30141042afda6863f20d](https://github.com/user-attachments/assets/9d0a510f-51a0-4555-b9c7-8fcc8b1f5197)
  
   * **EXT4 oder LVM** (Standard) geht ebenfalls, ist simpler, aber ohne fortgeschrittene Features (Snapshots, Checksums).

3. **Land/Zeitzone/Tastatur**

![02b2ddf50e5821de7c1e222fa4370adb](https://github.com/user-attachments/assets/93801a6c-f39c-4bf9-9a63-486c3bd57f27)

   * Besser direkt korrekt einstellen:

     * **Zeitzone** (z.B. Europe/Berlin),
     * **Keyboard Layout** (ger, en, etc.).

4. **Passwort und E-Mail**

![22c6c1261b1bf5043d80bdc52047ddd5](https://github.com/user-attachments/assets/cd2d044c-75da-451a-bc8f-d65aac10216d)

   * **Root-Passwort**: Wählen Sie ein sicheres (mind. 12 Zeichen, Kombination aus Buchstaben, Ziffern, Symbolen).
   * **E-Mail**: Dient für Benachrichtigungen (z.B. Backup-Ergebnisse, Cluster-Alerts).

5. **Netzwerkkonfiguration**

![5816ad5f7d3f31b41efc795903ebdf8b](https://github.com/user-attachments/assets/581a1a47-fa14-47d5-afb7-2977e16117f2)

   * IP-Adresse, Subnetzmaske, Gateway, DNS eintragen.
   * Hostname = `<nodename.domain.tld>` (z.B. `pve01.lab.local`).
   * **Achtung**: Bei Clustern muss jeder Node eine unique IP und Hostname haben. Keine Duplikate!

6. **Installation**

![02e58a172ac0e0dadab5483ef21105a4](https://github.com/user-attachments/assets/84e3c404-f646-44b1-9cfb-05b7a754c8f1)

   * Der Installer formatiert das Zielmedium, legt je nach Auswahl ZFS-Pool oder LVM-Partition an.
   * Dies kann einige Minuten dauern, je nach Geschwindigkeit des Speichermediums.

7. **Neustart**

   * USB-Stick abziehen, System bootet in das frisch installierte Proxmox VE.

***

## 2.3 Nach dem ersten Boot

Nach erfolgreichem Hochfahren loggen Sie sich erstmal **Lokal** oder via Weboberfläche ein:

* **Webbrowser**: `https://<IhreIP>:8006`
  * Da Proxmox VE ein selbstsigniertes Zertifikat erstellt, erhalten Sie meist eine Warnung im Browser; bestätigen und fortfahren.
* **Login**: `root` + Ihr vergebenes Passwort.

![2cbb0ca870c0a5c458285e58e7c25e12](https://github.com/user-attachments/assets/fbdc694e-6d44-4e20-82ed-5f3c5fe36ed9)

> **Tipp**: Oft ist der Port 8006 blockiert, wenn man die Firewall zu restriktiv eingestellt hat. Prüfen Sie ggf. die Firewall (lokal oder extern).

### 2.3.1 Enterprise vs. Community Repository

* Nach dem Login erscheint u.U. ein Hinweis, dass Sie keine Subscription haben, wenn Sie die Enterprise-Repos nutzen wollen. In einer Testumgebung können Sie das ignorieren oder die Repos in /etc/apt/sources.list.d/pve-enterprise.list auskommentieren und das Community-Repo ergänzen:

  ```bash
  # Enterprise-Repo auskommentieren
  sed -i "s/^deb /#deb /" /etc/apt/sources.list.d/pve-enterprise.list

  # Community-Repo anlegen:
  echo "deb http://download.proxmox.com/debian/pve bullseye pve-no-subscription" >> /etc/apt/sources.list

  apt-get update
  ```

### **2.3.2 Erstes Update**

Nach der erfolgreichen Installation von Proxmox VE ist es von entscheidender Bedeutung, das System auf den neuesten Stand zu bringen, um Sicherheitslücken zu schließen, die Stabilität zu verbessern und neue Funktionen zu integrieren. Proxmox VE bietet zwei Wege für die Aktualisierung: über die **GUI** und die **CLI**.

***

#### **Updates via GUI (empfohlen)**

Die grafische Benutzeroberfläche (GUI) von Proxmox VE bietet eine intuitive Möglichkeit, Updates durchzuführen.

1. **Repository-Überprüfung:**

   * Navigieren Sie zu **Datacenter → Updates**.

   * Vergewissern Sie sich, dass das korrekte Repository verwendet wird:

     * Standardmäßig ist das **Enterprise-Repository** aktiviert. Ohne gültige Subscription sollten Sie zum **Community-Repository** wechseln:

       * Gehen Sie zu **Node → Repositories**.

       * Wählen Sie das Repository `pve-enterprise` aus und deaktivieren Sie es.

       * Fügen Sie das **Community-Repository** hinzu:

         * Klicken Sie auf **Add**.
         * Wählen Sie `No-Subscription`.
      ![b740130a3b9c37f6fe0b49a34684cea7](https://github.com/user-attachments/assets/c48f086b-b048-4ef6-b052-4241d3f67cfe)
         * Klicken Sie auf Add**.

2. **System aktualisieren:**

   * Gehen Sie zu **Node → Updates**.
   * Klicken Sie auf **Refresh**, um die neuesten Paketlisten zu laden.
   * Klicken Sie auf **Upgrade**, um verfügbare Updates zu installieren.

3. **System neu starten:**

   * Nach der Installation von Kernel-Updates und anderen sicherheitskritischen Paketen ist ein Neustart erforderlich.
   * Klicken Sie auf **Node → System → Reboot**, um das System neu zu starten.

***

#### **Updates via CLI**

Für Nutzer, die die Kommandozeile bevorzugen, kann das Update auch direkt über SSH oder die lokale Konsole durchgeführt werden.

1. **SSH-Verbindung herstellen (falls nötig):**

   * Verbinden Sie sich mit dem Server über SSH:
     ```bash
     ssh root@<IP-Adresse>
     ```

2. **Repository-Einstellungen anpassen:**

   * Deaktivieren Sie das Enterprise-Repository (falls keine Subscription vorliegt):
     ```bash
     sed -i "s/^deb /#deb /" /etc/apt/sources.list.d/pve-enterprise.list
     ```
   * Fügen Sie das Community-Repository hinzu:
     ```bash
     echo "deb http://download.proxmox.com/debian/pve bullseye pve-no-subscription" >> /etc/apt/sources.list
     apt-get update
     ```

3. **Updates installieren:**

   * Führen Sie das Upgrade durch:
     ```bash
     apt-get dist-upgrade -y
     ```
   * Dabei werden alle verfügbaren Updates installiert.

4. **Neustart durchführen:**

   * Nach Kernel-Updates ist ein Neustart erforderlich:
     ```bash
     reboot
     ```

***

#### **Wichtige Hinweise**

1. **Regelmäßige Updates:**

   * Aktualisieren Sie Ihr System regelmäßig, um sicherzustellen, dass Sie stets die neuesten Sicherheits-Patches und Bugfixes erhalten.
   * Planen Sie Updates in Wartungsfenstern ein, insbesondere wenn Downtimes kritisch sind.

2. **Backup vor Updates:**

   * Führen Sie vor größeren Updates ein Backup wichtiger Daten oder VMs durch, um bei Problemen auf einen vorherigen Zustand zurückgreifen zu können.

3. **Überwachung des Update-Prozesses:**

   * Während des Updates zeigt Proxmox in der GUI detaillierte Logs an. Beobachten Sie den Fortschritt und überprüfen Sie, ob alle Pakete erfolgreich aktualisiert wurden.

***

Mit dieser Anleitung stellen Sie sicher, dass Ihre Proxmox VE Installation sicher, stabil und auf dem neuesten Stand bleibt. Der GUI-Ansatz ist besonders nutzerfreundlich und bietet einen schnellen Überblick, während die CLI für Automatisierungen oder detaillierte Kontrolle bevorzugt wird.


### 2.3.3 Zeitsynchronisierung

* In der GUI: _Datacenter_ → _Node_ → _System_ → _Time_.
* Konfigurieren Sie NTP-Server (z.B. pool.ntp.org oder interne Zeitserver).
* Gerade bei Clustern oder Snapshots können falsche Zeiten zu großen Problemen führen (z.B. abgelehnte Zertifikate, HA-Failover-Fehler).

### 2.3.4 Netzwerkprüfung

* _Datacenter_ → _Node_ → _System_ → _Network_:

  * Prüfen Sie, ob die IP, Gateway, DNS korrekt sind.
  * Wollen Sie VLAN Aware oder Bonding, können Sie es direkt hier anlegen.
  * Vorsicht, das Speichern falscher Einstellungen kann den Host vom Netz trennen.

***

## 2.4 Partitionierungs- und Storage-Überlegungen bei der Installation

### 2.4.1 ZFS (Root-FS)

* Wenn Sie beim Installer **ZFS** gewählt haben, existiert nun z.B. ein Pool `rpool`.
* Vorteil: Snapshots auf Root-Ebene, Checksums, Mirror/RAIDZ.
* Nachteil: Höherer RAM-Verbrauch, potenziell mehr Konfigurationsaufwand bei Updates.

### 2.4.2 LVM / EXT4

* Das Standardlayout von Proxmox bei „Directory“ + LVM kann ok sein, wenn man nicht zwingend ZFS-Features möchte.
* Ggf. legen Sie später separate Datenträger für VMs an – z.B. extra SSD, die als LVM-Storage oder ZFS-Pool fungiert.

### 2.4.3 Nachträgliche Änderungen

* Falls Sie ohne ZFS installiert haben, aber später ZFS möchten, ist meist ein Neuinstall oder manuelles Einbinden eines zweiten Pools nötig. Planung im Vorfeld erspart hier Schmerzen.

***

## 2.5 Häufige Stolperfallen (Installation)

1. **BIOS vs. UEFI**

   * Viele User wundern sich, warum kein Boot vom USB-Stick klappt. Secure Boot kann es verhindern.
   * Ein Umstellen auf Legacy oder UEFI-Mode kann Abhilfe schaffen.

2. **APIC- oder VT-x im BIOS deaktiviert**

   * Manche Mainboards haben standardmäßig Virtualisierung abgeschaltet. Ohne VT-x/AMD-V startet KVM nur eingeschränkt.
   * Im BIOS/UEFI: Optionen „Intel Virtualization Technology“ oder „SVM Mode“ aktivieren.

3. **Falsches Partitionierungsschema**

   * Wer manuell in parted/GPT rummanipuliert, kann den Installer durcheinanderbringen.
   * Besser dem Proxmox-Installer die Kontrolle über die gewählte Platte geben oder alles vorher manuell löschen (z.B. `wipefs -a /dev/sdX`).

4. **Binden an DHCP**

   * Wenn man Node mit DHCP ausliefert, kann es sein, dass IP wechselt und man die WebGUI nicht mehr findet.
   * Lösung: Statische IP oder DHCP Reservation.

5. **Nicht auskommentierte Enterprise-Repo**

   * User wundern sich über lästige Subscription-Meldungen oder Update-Fehler.
   * Entweder Subscription kaufen oder das Repo auskommentieren und Community-Repo eintragen.

***

## 2.6 Erste VM / Container als Test

**Empfehlung**: Sobald das System steht, und Sie die SSH- bzw. WebGUI-Zugänge getestet haben, probieren Sie Folgendes:

1. **ISO hochladen**

   * Datacenter → Node → Local (oder eigenes Storage) → Content → Upload.
   * Wählen Sie z.B. ein Ubuntu-Server-ISO.

2. **VM anlegen**

   * Klicken Sie auf _Create VM_, geben Sie Name, ISO, Storage an.
   * CPU/RAM je nach Geschmack (z.B. 2 Cores, 2 GB RAM).
   * Netzwerk: Standard: _vmbr0_, bridged Modus.

3. **Installation**

   * Starten Sie die VM, wählen Sie in der Konsole (noVNC, Spice) das ISO.
   * Beobachten Sie Performance, Bootzeiten, Logs.

4. **Snapshot**

   * Nach dem OS-Setup einen _Snapshot_ anlegen, um die Snapshot-Funktionalität zu erproben.
   * ZFS-Root, LVM-Thin oder Directory? Je nach Storage kann das Feature mehr oder weniger komfortabel sein.

***

# **Kapitel 3 – Storage-Konfiguration (ZFS, BTRFS, EXT4) und Best Practices**

Nachdem Sie Proxmox VE erfolgreich installiert haben, steht als Nächstes die **Einrichtung eines leistungsfähigen und verlässlichen Storages** an. Hier treffen wir richtungsweisende Entscheidungen über **Dateisystem**, **RAID-Level** und **Funktionsumfang** (Snapshots, Replikation, Kompression usw.). Je nachdem, wie Ihr zugrunde liegendes Laufwerks-Setup aussieht (einzelne Platten, Hardware-RAID, SSD/NVMe, HDDs), können Sie verschiedene Optionen wählen.

## 3.1 Warum ist der Storage so wichtig?

1. **Performance-Auswirkungen**

   * Virtuelle Maschinen laden/lesen Daten von der Host-Festplatte oder einem externen Storage. Die I/O-Geschwindigkeit (Input/Output) entscheidet oft über die gefühlte Geschwindigkeit der VMs. Eine schlecht performende Platte kann VMs verlangsamen oder zum Engpass werden.
   * ZFS oder BTRFS haben Funktionen wie Copy-on-Write, Kompression, Checksums, die einerseits Daten schützen, andererseits mehr Ressourcen beanspruchen.

2. **Datenintegrität**

   * **ZFS** ist bekannt dafür, sehr gute Datenintegrität zu bieten, indem es Checksums anlegt und defekte Blöcke selbstständig erkennen und ggf. reparieren kann.
   * RAID-Verbünde (z.B. RAID-Z oder Mirror) reduzieren das Risiko eines Datenverlusts bei Laufwerksausfällen.

3. **Snapshot- und Replikations-Funktionen**

   * Container oder VMs lassen sich per Snapshot sichern oder klonen. Ein gutes Dateisystem erleichtert das und verbraucht dabei wenig Platz (Copy-on-Write).
   * Replikation – also das Kopieren dieser Snapshots an andere Standorte oder Knoten – ist ein essenzieller Baustein in HA- oder Backup-Strategien.

***

## 3.2 Übersicht der gängigen Dateisysteme

### 3.2.1 ZFS (Zettabyte File System)

* **Merkmale**:

  * Copy-on-Write-Prinzip: Anstelle, dass Änderungen direkt in den Originalblock geschrieben werden, kreiert ZFS neue Blöcke. Alte Daten bleiben erhalten, was Snapshots sehr effizient macht.
  * Integrierte **RAID-Level** (RAID-Z1, Z2, Z3, Mirror), Kompression (lz4, zstd), Deduplication (optional).
  * Starke Datenintegrität via Checksums.

* **Vorteile**:

  1. **Hohe Verlässlichkeit**: Jede Schreiboperation wird mit einer Prüfsumme gesichert.
  2. **Reiche Feature-Palette**: Snapshots, Replikation, Send/Receive.
  3. **Skalierbar**: Hinzufügen oder Erweitern von Pools, Cache-Geräte (SLOG, L2ARC).

* **Nachteile**:

  1. Braucht **mehr RAM** (Faustregel: 1 GB RAM pro TB Storage).
  2. Komplexer RAID-Mechanismus, man sollte die RAID-Z-Variante bewusst wählen (RAIDZ1 bei >=3 Disks, RAIDZ2 bei >=4 Disks usw.).

* **Wer sollte ZFS wählen?**

  * Wer **Snapshots** oft und gerne nutzt, auf Checksums und höchste Datenintegrität Wert legt.
  * Wer z.B. Container und VMs in produktionsnaher Qualität betreiben will.

### 3.2.2 BTRFS

* **Merkmale**:

  * Copy-on-Write, Subvolumes, native Snapshots, rudimentäre RAID-Funktionen (RAID0, 1, 10, experimentell 5/6).
  * BTRFS kann auf einem einzigen Blockdevice laufen und verschiedene Subvolumes bereitstellen.

* **Vorteile**:

  1. **Simpler Einstieg**: Auf einer einzigen Partition kann man mehrere Subvolumes schaffen.
  2. **Snapshots**: Schnell und effizient durch Copy-on-Write.
  3. **Kompression**: Optional aktivierbar (z.B. zlib, lzo).

* **Nachteile**:

  1. RAID5/6 gilt als **nicht 100% stabil**. Die Community warnt häufig vor produktivem Einsatz.
  2. Weniger verbreitet in Proxmox-Umgebungen als ZFS (geringere Erprobung).

* **Einsatzgebiet**:

  * Gut für Einzelplattensysteme mit Snapshots, wenn man weniger RAM einsetzen will und nicht zwangsläufig RAIDZ-Features braucht.

### 3.2.3 EXT4

* **Merkmale**:

  * Klassisches Linux-Dateisystem, sehr stabil, ressourcenschonend, weit verbreitet.
  * Keine nativen Snapshots oder integrierte RAID-Funktionen (dafür braucht man LVM oder ein Hardware-RAID).

* **Vorteile**:

  1. **Vertrautheit**: Die meisten Linux-User kennen EXT4; Debugging und Tools sind ausgereift.
  2. **Geringe Overhead**: Braucht kein zusätzliches Memory- oder CPU-Budget für Checksums/Kompression.

* **Nachteile**:

  1. Keine eingebauten Snapshots.
  2. Keine eingebaute Checksums – potenzielle still errors (bit rot) werden nicht automatisch erkannt.

* **Für wen?**

  * Wer auf einer einzelnen SSD/HDD ohne besondere Features (Snapshots, RAID im FS) klarkommen will.
  * Wer die Einfachheit schätzt und Snapshots/RAID z.B. via LVM oder Hardware-Controller macht.

***

## 3.3 ZFS im Detail (Pool-Konzepte, Cache, Snapshots)

### 3.3.1 Pool-Konzepte und RAID-Level

1. **RAID-Z1**: Mit mind. 3 Platten, eine Paritätsplatte. Vergleichbar mit RAID5, ein Plattenausfall ist tolerierbar.
2. **RAID-Z2**: Mind. 4 Platten, zwei Paritäten. Vergleichbar mit RAID6. Höhere Ausfallsicherheit.
3. **RAID-Z3**: Mind. 5 Platten, drei Paritäten. Selten in kleinen Umgebungen, eher in großen Arrays.
4. **Mirror**: 2+ Platten spiegeln sich gegenseitig (ähnlich RAID1). Höhere Performance (Read) und schnelle Resilver-Zeiten, aber weniger nutzbarer Platz.

> **Best Practice**:
>
> * RAIDZ1 kann bei großen Disks (4 TB+) riskant sein, wenn eine Platte ausfällt und Resilver lange dauert. RAIDZ2 oder Mirror ist sicherer.
> * Planen Sie immer etwas Kapazitätsreserve ein. ZFS-Pools mit >80–85% Füllgrad verlieren Performance.

### 3.3.2 SLOG (Separate Intent Log) und L2ARC

* **SLOG**:

  * Für synchrone Schreibvorgänge speichert ZFS das ZIL (ZFS Intent Log) auf einer dedizierten, schnellen SSD/NVMe.
  * Resultat: Beschleunigtes Schreiben, wenn Clients z.B. NFS oder iSCSI mit sync=always nutzen.

* **L2ARC**:

  * Ein Lesecache auf SSD/NVMe. Häufig gelesene Daten können schneller ausgeliefert werden.
  * Aber: L2ARC selber beansprucht RAM, um Metadaten zu verwalten. Zu große L2ARC kann den Hauptspeicher belasten.

### 3.3.3 Snapshots & Replikation

1. **Snapshots**

   * `zfs snapshot poolname/dataset@snapname`.
   * In Proxmox-GUI kann man per „Snapshots“-Reiter einer VM Container/Volumes bequem anlegen.
   * Blitzschnelles Rollback ohne Kopieren riesiger Daten.

2. **ZFS Send/Receive**

   * `zfs send poolname/dataset@snap | ssh user@remoteserver "zfs receive remotepoolname/..."`
   * So kann man ganze Dateisysteme oder VMs an einen anderen Host übertragen, ideal für Offsite-Backups oder DR-Szenarien.

### 3.3.4 Deduplication?

* ZFS unterstützt Deduplizierung, ist aber sehr RAM-intensiv (Hash-Tabellen). In Testumgebungen mit wenig RAM kann das System spürbar ausbremsen.
* Im Zweifel Kompression aktivieren (lz4, zstd) und Dedup weglassen, außer Sie haben sehr viel RAM und spezielle Anwendungsfälle (viele identische Datenblöcke).

***

## 3.4 BTRFS: Subvolumes, Snapshots und RAID

### 3.4.1 Subvolumes

* BTRFS ermöglicht, innerhalb einer Partition mehrere Subvolumes zu erstellen. Jedes Subvolume kann man unabhängig mounten oder einen eigenen Snapshot davon anlegen.
* Auch Container können pro Subvolume isoliert werden, was Container-Backups oder -Rollbacks erleichtert.

### 3.4.2 RAID-Funktionen

* RAID0, RAID1, RAID10 gelten als stabil.
* RAID5/6 gilt als experimentell; es gibt Berichte über instabile Wiederherstellungen bei Disk-Ausfällen.
* Performance: BTRFS kann solide sein, liegt in Benchmarks aber meist hinter ZFS, vor allem bei größeren Datensätzen und RAID5/6.

### 3.4.3 Integration in Proxmox

* BTRFS ist nicht der offizielle Standard, aber man kann z.B. ein BTRFS-Dateisystem als Directory-Storage in Proxmox einhängen. Snapshots muss man teils manuell über CLI (`btrfs subvolume snapshot ...`) verwalten.
* Vorsicht bei LXC, da manche BTRFS-Funktionen in Containern oder Host-BTRFS-Overlay weniger getestet sind.

***

## 3.5 EXT4: Klassische Stabilität

* EXT4 ist sehr ausgereift, Tools wie `fsck.ext4` sind verbreitet, Ausfallsicherheit kommt meist aus dem darunterliegenden LVM oder Hardware-RAID.
* Wer einfach in einer Testumgebung sehr **simpel** starten will, kann EXT4 nehmen und das Thema Snapshots via Proxmox-eigenen Mechanismen (LVM-Thin, Backup-Snapshots) lösen.
* Weniger overhead, dafür nicht die ZFS-typischen Checksums (bit rot Erkennung, etc.).

***

## 3.6 Einrichtung in Proxmox VE (GUI und CLI)

### 3.6.1 Storage hinzufügen (Allgemein)

1. **Proxmox-Weboberfläche** → _Datacenter → Storage → Add_

2. **Typ wählen**:

   * _Directory_ (EXT4/BTRFS gemountet),
   * _ZFS_ (ZFS-Pool),
   * _LVM_ (falls Sie LVM für VM-Images nutzen wollen),
   * _iSCSI_, _NFS_, _CIFS_ etc.

3. **ID**: Geben Sie einen eindeutigen Namen (z.B. `local-zfs`, `ext4-store`).

4. **Content**: Welche Inhalte? `Disk image`, `Container`, `ISO`, `VZDump backup`, etc.

5. **Speichern**, Proxmox lädt die Config und das Storage taucht in der Liste auf.

### 3.6.2 ZFS-Pool erstellen (GUI)

1. _Datacenter → Node → Disks → ZFS_ (je nach Proxmox-Version kann das Menü etwas variieren).

2. _Create ZFS_:

   * Namen vergeben (z.B. `rpool2` oder `zfs-pool-raidz`).
   * Disks auswählen (z.B. `sda`, `sdb`, `sdc`).
   * RAID-Level: RAIDZ1, Z2, Z3 oder Mirror.
   * Optional: Cache/SLOG-Device.

3. **Bestätigen**: Proxmox initialisiert den Pool. Achtung: vorhandene Daten auf den Disks werden gelöscht.

### 3.6.3 BTRFS oder EXT4 als Directory

* Wenn Sie z.B. manuell BTRFS/EXT4 formatieren, mounten Sie diese Partition unter `/mnt/<Verzeichnis>`.
* Proxmox: _Datacenter → Storage → Add → Directory_, `Directory: /mnt/<Verzeichnis>`, Content: `Disk image`, `Container`, etc.
* Nun lassen sich VMs/Container in diesem Ordner ablegen.

***

## 3.7 Best Practices

1. **Planen Sie redundante Disks**

   * Ein Single Disk kann bei Ausfall alles gefährden. RAID-Level (RAIDZ oder Mirror) oder ein externes SAN/NAS sind empfehlenswert.

2. **SMART-Überwachung**

   * Regelmäßig `smartctl -a /dev/sdX` checken, oder Monitoring einrichten, damit defekte Sektoren früh erkannt werden.

3. **RAM**

   * Gerade bei ZFS: Lieber etwas mehr RAM bereitstellen. RAM-Sättigung kann Performance stark beeinflussen.

4. **Snapshots automatisieren**

   * Proxmox eigner Backup-Job (VZDump) oder ZFS-Snapshots (via Cron/Script). Wenn man Snapshots häuft, sollte man sie auch aufräumen (Löschen alter Snapshots).

5. **Pool-Füllstand**

   * Achten Sie darauf, ZFS-Pools nicht über 80–85 % zu füllen. Sonst Einbußen bei Write-Performance, da Copy-on-Write mehr overhead hat.

***

## 3.8 Troubleshooting (Storage)

1. **Leistungseinbrüche**

   * `zpool iostat -v 2` – zeigt IOPS pro vdev. Engpässe bei einer Platte?
   * BTRFS: `btrfs device stats /mountpoint` – meldet z.B. IO-Fehler.

2. **ZFS „DEGRADED“**

   * Meist Platte ausgefallen. `zpool status -v <pool>` – defekte Disk replacen, resilvern lassen.

3. **EXT4-FSCK**

   * Muss offline (unmounted) oder readonly sein. Planen Sie Downtime ein.

4. **Nicht genug RAM für ZFS**

   * `ARC size` kann man in `/etc/modprobe.d/zfs.conf` beschränken (`options zfs zfs_arc_max=...`).
   * Bei sehr wenig RAM lieber EXT4 oder BTRFS.

> **Tipp**: Legen Sie sich einen kleinen „Testpool“ mit virtuellen Festplatten (z.B. auf einer anderen Platte) an, um Snapshots/Replications gefahrlos zu üben.

***

## 3.9 Zusammenfassung zu Storage

**Fazit**: Das Dateisystem und die Storage-Struktur sind zentrale Bausteine in Proxmox VE. ZFS bietet den größten Funktionsumfang, benötigt aber entsprechende Hardware-Ressourcen. BTRFS und EXT4 sind Alternativen, wenn man es schlichter halten will oder weniger RAM einsetzen möchte. Unabhängig vom FS gilt: Redundanz, regelmäßige Checks und Backups sind unerlässlich, um Datenverlusten vorzubeugen.

***

# **Kapitel 4 – Netzwerkkonfiguration**

_(Dieser Teil baut auf dem Storage-Kapitel auf. Nun stehen die Grundlagen zur Verfügung, um auch mehrere Knoten und VMs/Container über ein professionelles Netzwerk zu verbinden.)_

## 4.1 Warum Netzwerkkonfiguration so entscheidend ist

* **Performance**: Wenn Storage und Management über dasselbe Interface laufen, kann es Engpässe geben.
* **Sicherheit**: Management, VM, Container, DMZ – all das sollte man trennen (via VLAN oder physische Ports).
* **Clustering**: Corosync-Heartbeat erfordert zuverlässiges, möglichst latenzarmes Netzwerk. Fehlerhafte Switch-Einstellungen (z.B. Spanning Tree Pausen) können HA-Entscheidungen verfälschen.

## 4.2 Bridging in Proxmox VE

### 4.2.1 Grundidee

Eine „Bridge“ in Linux/Proxmox agiert wie ein virtueller Switch. Wenn `vmbr0` an `eth0` hängt, können VMs/Container direkt ins physische Netz. IPv4/IPv6-Adressen konfiguriert man meist auf `vmbr0`, nicht mehr auf `eth0`.

* **Vorteil**: VMs erhalten echte IP-Adressen (keine NAT). Das erleichtert den Zugriff.

* **Setup**:

  1. _Datacenter → Node → System → Network_
  2. _Create → Linux Bridge_
  3. _Bridge ports:_ z.B. `eth0`.
  4. IP-Config auf die Bridge legen, NIC selbst bleibt häufig ohne IP.

### 4.2.2 VLAN Aware

* Aktiviert man „VLAN Aware“ in der Bridge, kann man innerhalb einer VM/Container VLAN-Tags verwenden. So lassen sich z.B. VLAN 10, 20, 30 an einer physischen NIC trunken und in VMs verteilen.

> **Beispiel**:
>
> * `vmbr0` (VLAN Aware) an `bond0`.
> * VM A legt VLAN=20 am Gastinterface an, VM B VLAN=30. Der Switch dahinter muss dieselben VLAN IDs trunken.

***

## 4.3 Netzwerk-Bonding

### 4.3.1 Ausfallsicherheit und Bandbreite

**Bonding** (Teaming) fasst mehrere physische Ports zusammen:

* **Mode 4 (802.3ad)** LACP:

  * Erfordert Switch-Konfig (Aggregierter LACP-Port).
  * Gute Lastverteilung je nach Hash-Policy.

* **Active-Backup (Mode 1)**:

  * Eine Karte aktiv, die andere in Standby. Einfachste Ausfallsicherheit, kaum extra Bandbreite.

* **balance-rr (Mode 0)**:

  * Round-Robin-Paketzuteilung. In reinen Test-Setups, wenn Switch kein LACP beherrscht. Kann jedoch bei hohen Lasten ungewollte Reorder-Effekte erzeugen.

### 4.3.2 Umsetzung in der GUI

1. _Datacenter → Node → System → Network_
2. _Create → Linux Bond_
3. Bond-Slaves auswählen (z.B. `enp3s0`, `enp4s0`), Modus 802.3ad, miimon=100, etc.
4. Bond-Device (z.B. `bond0`) per Bridge (vmbr0) sichtbar machen.

### 4.3.3 Switch-Seite

* Switch-Port muss auf LACP/Trunk konfiguriert sein, wenn Sie Mode 4 wollen.
* Jede NIC in denselben LAG-Portchannel. VLAN-Settings passend trunken.
* Prüfen Sie Link-Status via `ip a` oder `cat /proc/net/bonding/bond0`.

***

## 4.4 VLAN-Konfiguration

### 4.4.1 Ziel der VLANs

* Logische Netzwerksegmentierung, z.B.:

  * VLAN 10: Management
  * VLAN 20: Produktions-VMs
  * VLAN 30: Storage/iSCSI
  * VLAN 40: DMZ

So minimieren Sie Broadcast-Overhead, erhöhen Sicherheit (Angreifer auf VM-Netz kann nicht einfach ins Management-Netz).

### 4.4.2 VLAN Aware Bridge vs. Linux VLAN Interface

1. **VLAN Aware Bridge**

   * Haken bei `VLAN Aware` im Bridge-Interface setzen.
   * VM kann VLAN-Tag direkt im Gast angeben, oder man konfiguriert in Proxmox, dass die VM-Port VLAN 20 hat.

2. **„Linux VLAN“** (z.B. `vlan10`)

   * Proxmox kann ein Interface `vlan10` an `bond0` erstellen, IP zuweisen.
   * Dann an `vmbr1` hängen. Etwas komplexer, aber klarere Trennung auf Host-Ebene.

> **Praxis-Tipp**:
>
> * In vielen Setups ist `vmbr0 (VLAN Aware) → bond0 (Trunk)` üblich. VMs werden mit einer VLAN ID verknüpft.
> * Achten Sie darauf, dass Switchport auf trunk mode für VLAN 10, 20, 30 konfiguriert ist.

***

## 4.5 Firewall-Architektur

### 4.5.1 Interne Proxmox-Firewall

* Proxmox-eigene Firewall kann auf Datacenter-, Host- oder VM-Ebene Regeln definieren.
* Globale Policies: Standard-Drop für WAN, nur 22/8006 auf definierte IPs.
* VM-spezifische Firewalls: z.B. Container darf nur bestimmte Ports inbound akzeptieren.

### 4.5.2 Externe Firewall

* Häufig pfSense/OPNsense oder dedizierte Hardware (Cisco, Sophos, Fortinet).
* Vorteile: Einfache zentrale Regelverwaltung, NAT/VPN an einem Ort.
* Viele nutzen eine **Firewall-VM** (pfSense) auf Proxmox selbst. Dann VLAN trunk ins WAN, VLAN trunk ins LAN, pfSense regelt Zugriffe.

***

## 4.6 DHCP vs. Statische IPs

* **Statisch**: Sinnvoll für Management-Schnittstellen, da IP-Wechsel ungewünschte Überraschungen provoziert.
* **DHCP**: Kann man in VMs (Gäste) nutzen, um dynamische Adressen zu vergeben.
* **Reservierungen**: In produktionsnahen Testlabs kann man DHCP-Reservierungen definieren (feste IP pro MAC), um eine Mischform zu haben.

***

## 4.7 Netzwerk-Performance und Tuning

1. **MTU / Jumbo Frames**

   * Erhöhen auf 9000, wenn ALLE Switches und NICs Jumbo Frames unterstützen.
   * Gute Effekte beim Storage-Traffic (z.B. iSCSI/NFS).
   * Größere Pakete = weniger Overhead, aber nur sinnvoll, wenn lückenlos durchgängig konfiguriert.

2. **RSS (Receive Side Scaling), TSO (TCP Segmentation Offload)**

   * Standardmäßig an. Check mit `ethtool -k eth0`.
   * Kann CPU-Last bei hohen Durchsätzen abfedern.

3. **Bonding-Hash-Policy**

   * LACP hat _layer2+3_ oder _layer3+4_-Optionen. Je nach Switch-Einstellungen kann man NetFlow-Hash verteilen.

4. **Monitoring**

   * Tools wie `iftop`, `nload` auf Host-CLI, oder deeper mit Prometheus/Grafana.
   * Erkennen Sie Engpässe (z.B. 1 Gbit/s Interface gesättigt).

***

## 4.8 Typische Stolperfallen (Netzwerk)

1. **Bridge/IP-Konfiguration verwechselt**

   * IP nicht auf `eth0` legen, sondern auf `vmbr0`, wenn `eth0` in `vmbr0` Bridge port ist. Sonst verliert man Connectivity.

2. **Fehlende Switch-Konfig bei Bonding**

   * Man stellt Mode 4 (LACP) in Proxmox ein, hat aber Switch-Port nicht als LACP konfiguriert. Link fehlerhaft.
   * Oder man nutzt balance-rr, was Switch-seitig Misskonfiguration verursachen kann.

3. **VLAN-Tagging**

   * Vergisst VLAN Aware in Proxmox, Switchports trunken VLAN 10, 20 – VMs empfangen keine DHCP, kein Ping.
   * Teilweise STP (Spanning Tree) kann Ports blocken, wenn sie VLAN-Änderungen detektieren.

4. **DHCP-Ausfall**

   * Node verliert IP, oder bekommt neue IP. WebGUI ist nicht mehr erreichbar.
   * Tendenziell lieber manuelle IP für Management.

***

## 4.9 Zusammenfassung

Das Netzwerk bildet das Fundament für Managementzugänge, VM/Container-Traffic und Hochverfügbarkeits-Meldungen. Wer unbedacht an Bonding oder VLANs herangeht, kann schnell stundenlang debugging. Planen Sie Ihr Netzwerk-Layout klar:

1. Management-Interface (z.B. `vmbr0`) – statische IP.
2. VLANs für DMZ/Produktions-VMs, optional VLAN Aware.
3. Bei Bedarf Bonding (z.B. 802.3ad) für Ausfallsicherheit/Bandbreite.
4. Firewalls: interne oder externe – definieren Sie klar, welche Ports/Protokolle offen sind.

***

# **Kapitel 5 – Sicherheitsbest Practices in Proxmox VE**

Die Sicherheit einer Virtualisierungsplattform wie **Proxmox VE** kann gar nicht hoch genug eingestuft werden. Mit steigender Zahl von virtuellen Maschinen und Services wächst auch das Risikopotenzial – ein einziger kompromittierter Host kann die gesamte virtuelle Infrastruktur gefährden. In diesem Kapitel beleuchten wir **Sicherheitsgrundlagen**, **Härtungsmaßnahmen** und **moderne Tools** wie CrowdSec und Keycloak, um Single Sign-On (SSO) bereitzustellen.

## 5.1 Grundlagen der Sicherheit in Proxmox VE

### 5.1.1 Mehrschichtige Absicherung

1. **Netzwerkschicht**

   * Firewalls (intern oder extern), VLAN-Trennung, ggf. Bonding für Hochverfügbarkeit.
   * Segmentieren Sie Management-Netz vom produktiven VM-Traffic.

2. **Host-Schicht**

   * SSH absichern (Port, KeyAuth, 2FA).
   * Regelmäßige Updates, Hardening (z.B. AppArmor).
   * Minimale Dienste laufen lassen, ungenutzte Services deaktivieren.

3. **Zugriffskontrolle**

   * RBAC (Rollenbasierte Zugriffskontrolle) → Kein generelles root-Zugangschaos.
   * 2FA, Keycloak/LDAP/AD für Benutzerverwaltung.
   * Logging/Audit, um Zugriffsversuche zu dokumentieren.

4. **Intrusion Detection**

   * CrowdSec oder Fail2Ban, IDS (Suricata, Snort), ggf. pfSense/OPNsense, um Angriffe frühzeitig zu erkennen.

5. **Physische Sicherheit**

   * Abschließbarer Serverraum, USV, redundante Netzwerkpfade.

### 5.1.2 Proxmox-spezifische Aspekte

* Proxmox VE setzt auf Debian, also gelten dieselben Linux-Hardening-Prinzipien (z.B. `/etc/sysctl.conf`, SSH, iptables/nftables).
* Die Web-GUI läuft standardmäßig auf Port 8006, via HTTPS. Sie können diese Zugänge weiter beschränken (z.B. nur erlaubte IP-Bereiche).
* Container (LXC) bieten eine weniger starke Isolierung als VMs (KVM); je nach Sicherheitsanforderung kann ein Container also riskanter sein als eine komplette VM.

***

## 5.2 Firewall & Netzwerk-Absicherung

### 5.2.1 Integrierte Proxmox-Firewall

1. **Aktivierung**

   * _Datacenter_ → _Firewall_ → _Enable_ (oder pro Node).
   * Dann kann man globale (Datacenter-Level) oder Node- bzw. VM-spezifische Regeln definieren.

2. **Regeltypen**

   * **Accept**, **Drop**, **Reject**.
   * Inbound (Direction = In) oder Outbound (Direction = Out).
   * Zusätzliche Filter (Source, Destination, Ports).

3. **Typische Beispielregeln**

   * `SSH nur von 192.168.10.0/24` → _Action: Accept, Source: 192.168.10.0/24, Dest. Port: 22, Direction: In_.
   * `WebGUI nur intern` → _Action: Accept, Source: 192.168.50.0/24, Dest. Port: 8006_.

4. **Default Drop**

   * Alles andere blocken.
   * Achtung auf Whitelist von wichtigen Ports (22, 8006, ICMP?), sonst sperren Sie sich selbst aus.

> **Hinweis**: Wenn Sie Node-Firewall aktivieren, kann es sein, dass VM-spezifische Firewall-Regeln zusätzlich wirken (verschachtelt). Passen Sie die Reihenfolge an und testen Sie die Erreichbarkeit.

### 5.2.2 Externe Firewall (pfSense, OPNsense, Hardware)

* **Vor- und Nachteile**

  * _Pro_: Ein zentrales Regelwerk, kein Durcheinander in mehreren Host-Firewalls.
  * _Contra_: Ein Ausfall dieser einen externen Firewall kann das ganze Netzwerk lahmlegen (SPOF), es sei denn, man nutzt pfSense HA.

* **Typisches Setup**
  * WAN → (pfSense) → LAN. Proxmox-Hosts im LAN. Proxmox-Management port 8006 nur per VPN oder ACLs auf pfSense zugelassen.

* **Best Practices**

  * VPN-Zugriff (OpenVPN, WireGuard) für Administratoren, anstatt Ports ins Internet offen zu lassen.
  * IP- oder GeoIP-Block-Listen (wenn nur europäischer Zugriff notwendig).

***

## 5.3 SSH-Härtung

SSH ist einer der häufigsten Einfallstore, wenn standardmäßiger Port 22 offen ist und schwache Passwörter genutzt werden.

### 5.3.1 Port-Änderung

* In `/etc/ssh/sshd_config`:
  ```bash
  Port 2222
  ```
* `systemctl restart ssh`.
* Angreifer-Scanner, die nur auf Port 22 zielen, verpuffen. Natürlich kein Allheilmittel – geübte Angreifer scannen alle Ports.

### 5.3.2 Key-basierte Authentifizierung

1. **Clientseitig**

   * `ssh-keygen -t ed25519` oder `rsa -b 4096`.
   * Schlüssel im `~/.ssh/id_…` ablegen, ggf. Passphrase.

2. **Serverseitig**

   * `ssh-copy-id -i ~/.ssh/id_ed25519.pub -p 2222 root@proxmoxhost`.
   * `/etc/ssh/sshd_config`:
     ```
     PasswordAuthentication no
     PermitRootLogin prohibit-password
     ```
   * `systemctl restart ssh`.

3. **Vorteile**

   * Keine Passwörter (außer ggf. Key-Passphrase). Brute-Force extrem erschwert.
   * Kombinierbar mit Fail2Ban oder CrowdSec, um IPs bei Fehlversuchen zu sperren.

### 5.3.3 Root-Login-Einschränkung

* `PermitRootLogin no`

  * Alternativ: `PermitRootLogin without-password` oder `prohibit-password`.
  * Sinnvolle Praxis: Legen Sie einen normalen Benutzer (z.B. `adminuser`) an, geben Sie ihm `sudo`-Zugriff. Root-Login direkt nur im Notfall.

### 5.3.4 Zusätzliche Mechanismen

* IP-Restriktionen (z.B. in der Proxmox-Firewall: `Direction: In, Source: 192.168.x.x` only).
* Tools wie **CrowdSec** (siehe unten) oder **Fail2Ban** sperren IPs nach mehreren Fehlversuchen.

***

## 5.4 Zwei-Faktor-Authentifizierung (2FA)

Gerade bei GUI-Logins oder per SSH ist **2FA** (Time-based One-Time Password, TOTP) ein wirkungsvolles Mittel gegen Passwortdiebstahl.

### 5.4.1 Aktivierung in Proxmox

* In der Weboberfläche: _Datacenter → Permissions → Users_, gewünschten User -> _Edit_.
* Feld „**Two Factor**“ aktivieren, z.B. `TOTP`. Ein QR-Code erscheint, den Sie mit einer Authenticator-App (Authy, Google Authenticator, etc.) scannen.
* Bei Login: zusätzlich zum Passwort den zeitbasierten Token eingeben.

### 5.4.2 Pflicht für Admin-Accounts

* Insbesondere `root@pam` sollte 2FA nutzen, um unautorisierten Zugang zu erschweren.
* Achten Sie aber auf Notfallzugänge (Stichwort: Was, wenn Handy/Token weg?). Legen Sie ggf. Backup-Codes an oder haben Sie eine alternative local–Linux-Console-Zugriffsmöglichkeit.

***

## 5.5 CrowdSec

**CrowdSec** geht über das klassische Fail2Ban hinaus, indem es eine **Community-basierte Threat-Sharing-Plattform** bietet. Verdächtige IPs oder Verhaltensmuster werden zentral gesammelt, und Sie können automatisch profitieren, indem Sie globale IP-Blocklisten erhalten. Weitere Informationen finden Sie in der offiziellen Dokumentation unter <https://doc.crowdsec.net/>.

### 5.5.1 Installation und Konfiguration

```bash
curl -s https://packagecloud.io/install/repositories/crowdsec/crowdsec/script.deb.sh | bash
apt-get install crowdsec
systemctl enable crowdsec
systemctl start crowdsec
```

* Unter `/etc/crowdsec/` liegen Konfigurationsdateien.
* Definieren Sie in `/etc/crowdsec/acquis.yaml`, welche Logfiles verfolgt werden (z.B. `/var/log/auth.log`, `/var/log/pve/tasks/*.log`).
* Für eine einfache Integration in Proxmox kann das Paket `fulljackz/proxmox-bf` verwendet werden, das speziell zur Verbesserung der Brute-Force-Abwehr auf Proxmox-Systemen entwickelt wurde.

### 5.5.2 Funktionsweise

* CrowdSec scannt Logs auf wiederholte fehlgeschlagene SSH-Logins, Web-Zugriffe usw.
* Bei Erkennung bösartiger Patterns (Brute-Force, DDoS, etc.) wird die IP in der lokalen `decisions`-Liste blockiert (via iptables/nftables oder Proxmox-Firewall).
* Parallel kann CrowdSec mit dem globalen Backend kommunizieren: IPs, die andernorts als malicious erkannt wurden, sperrt es proaktiv.

### 5.5.3 Verwaltung

* `cscli decisions list` – zeigt blockierte IPs.
* `cscli decisions delete <ID>` – hebt Bann auf.
* Integration in Proxmox-Firewall: So bleiben Ban-Regeln konsistent, auch bei Reboots.

## 5.6 RBAC (Rollenbasierte Zugriffskontrolle)

**RBAC** ermöglicht feingranulare Zugriffsrechte: Nicht jeder darf alles.

### 5.6.1 Prinzip

1. **User**: Kann lokal (`pve`) oder über LDAP/AD/Keycloak angelegt sein.
2. **Rolle**: Definiert, welche Aktionen erlaubt sind (VM.Allocate, VM.Audit, Datastore.Allocate, etc.).
3. **Erstellung**: _Datacenter → Permissions → Roles._
4. **Zuweisung**: _Datacenter → Permissions → Add._ Geben Sie Pfad (`/`, `/vms/100`) an, den User, die Rolle.

### 5.6.2 Beispiel-Szenarien

* **Admin**: Vollzugriff auf alles.
* **VM-Manager**: Darf neue VMs anlegen, löschen, aber keine Cluster-Einstellungen ändern.
* **Backup-Operator**: Darf Backup-Jobs starten, Restores durchführen, nicht mehr.
* **Reine Lese-Rechte**: VM.Audit, Datastore.Audit, aber keine Schreiboperationen.

### 5.6.3 Pflege & Sicherheit

* Entfernen Sie veraltete Konten (Ex-Mitarbeiter etc.).
* Minimales Prinzip: Geben Sie so wenig Rechte wie möglich. Jemand, der nur Logs checken soll, kriegt nur Audit-Rechte.

***

## 5.7 Keycloak als SSO-Lösung

In Test- und auch Produktionsumgebungen kann es angenehm sein, **Single Sign-On** (SSO) via **Keycloak** zu nutzen, anstatt mehrere Authentifizierungsmethoden (lokal, AD, etc.) zu pflegen.

### 5.7.1 Vorteile

1. **Zentrale Identity-Verwaltung**: Keycloak kann AD/LDAP-Anbindung oder Social-Logins.
2. **2FA**: Keycloak selbst kann OTP/WebAuthn erzwingen, Proxmox muss es nicht gesondert regeln.
3. **Rollenmapping**: Keycloak-Gruppen → Proxmox-RBAC.

### 5.7.2 Implementierung (Kurzüberblick)

1. **Keycloak-Server**: z.B. Docker-Container oder separate VM.
2. **OpenID Connect Plugin** in Proxmox: Community-lösung, die OIDC realisiert.
3. **Realm**, **Client** in Keycloak anlegen: `redirect URIs`, `client secret`, etc.
4. **pvesh** (CLI) oder GUI, um OIDC-Auth-Provider hinzufügen.
5. **Nutzung**: Login-Flow leitet zur Keycloak-Anmeldeseite um, bei Erfolg geht’s zurück in die Proxmox-GUI.

### 5.7.3 Reverse-Proxy-Variante

* Alternativ: Setzen Sie z.B. `oauth2-proxy` oder Traefik als Forward-Auth.
* Proxmox-UI wird hinter einem Gateway angeboten. Wer nicht eingeloggt ist, wird zu Keycloak geleitet.
* Etwas komplexer, da Proxmox nativ kein SSO-Header-Parsing unterstützt, man trickst via proxied requests.

***

## 5.8 Weitere Sicherheitslayer

### 5.8.1 IDS/IPS: Suricata, Snort

* Erkennung von Netzwerk-Anomalien, Exploit-Signaturen.
* Häufig integriert in pfSense/OPNsense.
* Sinnvoll, wenn Proxmox-Hosts direkter Internetzugang oder exponierte Dienste haben.

### 5.8.2 AppArmor/SELinux

* Standardmäßig ist AppArmor auf Debian/Ubuntu-basierten Systemen präsenter, SELinux auf RHEL-basierten.
* Zusätzliche Sicherheitsprofile für PVE-Dienste, LXC, Qemu.
* Achtung: Ein fehlerhaftes Profil kann VMs oder Container blockieren. Testen Sie Konfigs sorgfältig.

### 5.8.3 Log-Überwachung (ELK, Graylog)

* Sammeln Sie Proxmox-Logs, SSH-Logs, Firewall-Ereignisse zentral, z.B. in Elasticsearch + Kibana.
* So erkennen Sie wiederholte Fehlanmeldungen oder seltsame Systemfehler.

### 5.8.4 Regelmäßige Updates

* `apt-get update && apt-get dist-upgrade -y` in regelmäßigen Intervallen (Maintenance-Fenster!).
* Achtung, Kernel-Updates erfordern Reboot → Planen Sie Downtimes ein oder nutzen Sie Cluster-Live-Migration, um VMs vorher auf andere Nodes zu schieben.

***

## 5.9 Zusammenfassung

Eine sichere Proxmox-Umgebung fußt auf mehreren Ebenen:

1. **Netzwerk**: Firewall-Regeln, VLAN-Segmentierung, kein offenes WAN-Interface ohne Schutz.
2. **Host-Härtung**: SSH-Port-Änderung, KeyAuth, 2FA, CrowdSec.
3. **RBAC & Keycloak**: Feingranulare Rechte, zentralisierte SSO.
4. **IDS/IPS & Logging**: Früherkennung von Angriffen, forensische Möglichkeiten.
5. **Ständige Pflege**: Updates, Log-Checks, Audit-Überprüfungen.

Damit ist das System gut gegen Einbrüche und Fehlbedienungen gerüstet.

***

# **Kapitel 6 – Clustering und Hochverfügbarkeit (HA)**

Während in den bisherigen Kapiteln die Basis (Installation, Storage, Netzwerk, Sicherheit) gelegt wurde, widmen wir uns nun dem **Mehrknotenbetrieb** und der **Hochverfügbarkeit** – dem entscheidenden Schritt in Richtung „produktionstaugliche“ Virtualisierungslösung. Ein Proxmox-Cluster ermöglicht:

1. **Zentrale Verwaltung** mehrerer Nodes über **eine** Weboberfläche.
2. **Live Migration** von VMs/Containern zwischen Knoten ohne (oder nur minimaler) Downtime.
3. **HA-Funktionalität**: Im Falle eines Node-Ausfalls können VMs/Container automatisch auf andere Knoten neu gestartet werden.

## 6.1 Cluster-Grundlagen

### 6.1.1 Corosync & Quorum

* **Corosync** ist der Messaging-Layer, der Node-Status (Heartbeat) und Cluster-Konfiguration synchron hält.
* **Quorum** bedeutet, dass mindestens die Hälfte +1 der Knoten (oder ein Quorum-Gerät) aktiv sein muss, damit Cluster-Entscheidungen getroffen werden dürfen (z.B. HA-Failover).
* Bei **zwei Knoten** ohne QDevice gibt es oft das „Split-Brain“-Problem: Beide glauben, sie seien der „letzte überlebende Knoten“. Deshalb empfiehlt man meist **≥3 Knoten** oder ein **QDevice** (Quorum-Device).

### 6.1.2 Shared Storage oder Replikation

* Für HA oder Live Migration benötigen VMs/Container ein **freigegebenes Storage**, das von allen Knoten aus erreichbar ist. Typische Varianten:

  1. **ZFS über iSCSI/NFS** oder **Ceph** (ein verteiltes Cluster-Dateisystem).
  2. **ZFS-Replikation**: VMs liegen auf lokalem ZFS, werden aber fortlaufend an andere Knoten repliziert. Bei Node-Failover kann man VMs dort aktivieren (etwas längere Downtime als bei echtem Shared Storage).

### 6.1.3 Netzwerk-Tipps

* Minimieren Sie Latenz und Paketverlust. Corosync mag es nicht, wenn Switch-Spanning-Tree Ports verzögert freischaltet oder VLANs instabil sind.
* Viele setzen ein dediziertes Interface (z.B. Bonding) für Cluster-Traffic (Heartbeat, Storage). So stört VM-Traffic nicht die Corosync-Pakete.

***

## 6.2 Cluster-Erstellung

### 6.2.1 Erster Knoten („Cluster Master“)

1. **Proxmox-Konsole** oder SSH:

   ```bash
   pvecm create <clustername>
   ```

   * Beispiel: `pvecm create pve-cluster`

2. **Corosync** startet automatisch, Status abfragen:

   ```bash
   pvecm status
   ```

   * Zeigt Quorum, Node-ID, Ringstatus an.

### 6.2.2 Beitritt weiterer Knoten

* Auf dem **zweiten** Node:

  ```bash
  pvecm add <IP-des-erstknotens>
  ```

  * Dann Root-Passwort eingeben, Corosync-Fingerprint prüfen und bestätigen.

* Alternativ Weboberfläche:
  * _Datacenter → Cluster → Join Cluster_ und die Join-Informationen (Fingerprint, IP, Key) ausfüllen.

### 6.2.3 Zwei Knoten vs. Drei Knoten

* **Zwei Knoten**: Man kann ein **QDevice** (Quorum-Device) einrichten, um das Split-Brain-Problem zu umgehen (zusätzlicher leichter Linux-Dienst, der als dritter „virtueller Knoten“ fungiert).
* **Drei (oder mehr) Knoten**: Sauberes Quorum. Häufigste Empfehlung für HA-Umgebungen.

***

## 6.3 Hochverfügbarkeits-Funktion (HA)

### 6.3.1 Idee

* Definieren Sie, welche VMs/Container „HA-Resources“ sein sollen. Fällt der Host-Knoten aus, erkennt das Cluster dies dank Corosync. Der HA-Manager startet die entsprechenden VMs/Container selbsttätig auf einem anderen, noch lebenden Knoten.

### 6.3.2 Voraussetzungen

1. **Gemeinsames Storage**:

   * VMs müssen physisch zugreifbar sein, sonst kann ein anderer Node nichts starten.
   * Entweder Ceph, NFS, iSCSI, oder ZFS-Sync.

2. **Stable Network**:
   * Ausreichende Corosync-Verbindung, sonst drohen Fehlentscheidungen.

### 6.3.3 Einrichtung

1. _Datacenter → HA → Groups_:
   * Erstellen Sie ggf. Gruppen (z.B. „prod-nodes“ = Node1, Node2).

2. _Datacenter → HA → Add_:

   * Wählen Sie VM/CT, ggf. die HA-Gruppe.
   * Service State: Enabled.

3. Prüfen via _Tasks_, ob Ressource aktiv und dem Cluster bekannt ist.

### 6.3.4 Failover-Vorgang

* Node stürzt ab oder reagiert nicht mehr → Corosync merkt „Node offline“, Quorum-Check.
* HA-Manager wartet den „**watchdog-timer**“ ab (60–120 s), um false positives zu vermeiden.
* Danach wird VM auf anderen Node neu gestartet. Minimale Ausfallzeit, je nach Storage/Netz.

> **Tipp**: Für Wartungszwecke _Datacenter → HA → Maintenance Mode_ oder manuell VM migraten, um sie sauber auf einen anderen Knoten zu verschieben.

***

## 6.4 Live Migration

### 6.4.1 Was ist Live Migration?

* Ein laufendes VM/Container-Gastsystem wird ohne signifikante Downtime vom Node A zu Node B verschoben:

  1. Proxmox kopiert den VM-RAM inkrementell.
  2. Finaler „Stop-and-copy“ ist sehr kurz (Sekundenbruchteile).
  3. Gast merkt evtl. minimalen Netzwerk-Ping-Verlust.

### 6.4.2 Voraussetzungen

* Gemeinsames Storage mit **Shared Disk Images**.
* Kompatible CPU-Flags (z.B. KVM64).
* Netzwerkverbindung mit ausreichendem Durchsatz für den RAM-Transfer.

### 6.4.3 Ablauf

1. WebGUI → VM → Migrate → Zielnode auswählen.
2. Proxmox zeigt Fortschrittsbalken.
3. Nach Abschluss läuft die VM auf dem Zielnode.

***

## 6.5 Georedundanz (Standortübergreifendes Clustering)

* Möchten Sie z.B. zwei Rechenzentren verbinden, kann man einen WAN-Cluster bauen.
* **Achtung**: Corosync reagiert sensibel auf Latenz und Paketverluste. Latenz >10–20 ms kann instabile Cluster-Situationen erzeugen.
* Häufig nutzt man asynchrone Replikation (ZFS oder Ceph) plus ein zweites Proxmox-Cluster, statt ein und denselben Cluster über riesige Distanzen zu spannen.

***

## 6.6 Typische Probleme & Troubleshooting

1. **Quorum-Verlust**

   * `pvecm status` → Quorum = 0%. Wenn >50% Knoten tot oder offline, kann Cluster nicht mehr HA-Entscheidungen treffen.
   * Notfall: `pvecm expected <Anzahl>`– nur in absoluten Ausnahmesituationen, weil Split-Brain droht.

2. **Split-Brain**

   * Node A und B sehen sich nicht, glauben beide, alleine „quorate“ zu sein. Potenziell Schreibkonflikt auf Shared Storage.
   * Minimieren durch 3+ Nodes, stabile Links, QDevice bei 2 Nodes.

3. **Fehlkonfiguration / Switch**

   * Bonding oder VLAN trunk fehlt → Corosync timeouts.
   * HA-Manager kann VMs blockieren (frozen state), weil Node-Communication fehlschlägt.

***

# **Kapitel 7 – Backup und Wiederherstellung**

Kommen wir zu einem der wichtigsten Themen: Datensicherheit. Eine Virtualisierungsplattform ohne **vernünftige Backup-Strategie** ist ein massiver Risikofaktor. Proxmox VE bietet eingebaute Tools wie VZDump, um VMs/Container zu sichern, zudem lassen sich externe Backup-Server (PBS, NFS, CIFS, etc.) nutzen.

## 7.1 VZDump – Proxmox Standard-Backup

### 7.1.1 Backup-Typen

1. **Snapshot-Mode**:
   * VM/Container wird kurz gesnapshottet, der dump erfolgt vom Snapshot. Minimale Downtime. Funktioniert gut auf LVM-Thin oder ZFS.
2. **Suspend**:
   * VM wird für die Backup-Dauer eingefroren (Suspend). Etwas längere Downtime, dafür verlässlich.
3. **Stop**:
   * VM erst herunterfahren, Backup erstellen, dann wieder starten. Längste Downtime, aber am sichersten was Datenkonsistenz angeht (bes. bei Datenbanken).

### 7.1.2 Backup-Speicher hinzufügen

* _Datacenter → Storage → Add → (Directory/NFS/CIFS/etc.)_.
* `Content: VZDump backup file`.
* Ggf. Offsite: z.B. NFS-Share eines externen NAS oder Cloud-Lösungen (rclone + lokal gemountet).

### 7.1.3 Backup-Jobs

* _Datacenter → Backup → Add_.

  * Node: Alle Knoten oder spezieller.
  * Schedule: Cron-Syntax (z.B. täglich 02:00 Uhr).
  * Auswahl: All VMs/Container, oder Liste.
  * Mode: Snapshot/Suspend/Stop.
  * Compression: LZO, GZIP, ZSTD.
  * Max backups: Alte Backups automatisch löschen.

### 7.1.4 Wiederherstellung

* _Datacenter → Storage → Content_: Wählen Sie die Backup-Datei (VMID, Datum) → _Restore_.
* Neuen VMID vergeben oder existierende VM überschreiben.
* Starten, prüfen Sie Log-Einträge und Funktionalität.

***

## 7.2 Offsite-Backup, Disaster Recovery

1. **Proxmox Backup Server (PBS)**:

   * Separates Produkt von Proxmox, dediziertes Dedupe-Backup.
   * Effiziente inkrementelle Dumps.
   * Gilt als robust und schlank.

2. **ZFS Send/Receive**

   * Fortlaufende Snapshots + send an einen anderen Node oder externes System. Ermöglicht schnelle DR-Szenarien.

3. **Cloud-Integrationen**

   * rclone + S3/Backblaze/Azure. Man kann VZDump-Files so extern lagern.

4. **Replikation**

   * Proxmox WebGUI: VM/Container → Replication. Wählt Quellnode und Zielnode, Intervalle.
   * Im Krisenfall kann man VMs am Zielnode starten, wenn Quellnode down ist.

***

## 7.3 Prüfung der Backups

* Nur ein **richtig getestetes** Backup ist wertvoll. Führen Sie z.B. monatlich eine Testwiederherstellung (stichprobenartig) durch, um sicherzugehen, dass die Dumpfiles intakt sind.
* Prüfen Sie Speicherauslastung: Werden alte Backups korrekt gelöscht?

***

# **Kapitel 8 – Monitoring und Wartung**

Kein System bleibt stabil ohne **Monitoring** und **regelmäßige Wartung**. Proxmox VE bietet eigene Grundübersichten zu CPU/RAM/Netz, aber umfangreichere Tools (Prometheus, Grafana, Zabbix, Nagios) lassen tiefere Einblicke zu.

## 8.1 Monitoring

### 8.1.1 Prometheus & Grafana

* **Prometheus**: Zeitreihen-Datenbank. Mithilfe eines Proxmox-Exporters oder Node-Exporter sammelt man Metriken (CPU, RAM, Platten-IO, Netz).
* **Grafana**: Visualisierung auf Dashboards. Zeigt z.B. Clusterübersicht, Auslastung pro Node, Container, VM.
* **Alarme**: Ab einer bestimmten CPU-Auslastung oder bei Plattenfehlern kann Grafana Alerts per Mail/Slack schicken.

### 8.1.2 Zabbix

* Eigenes Monitoring-System mit Agenten/Agentless-Optionen.
* Zabbix-Template für Proxmox existiert in der Community.
* Vorteil: Integriertes Eskalations- und Event-Handling, bewährt in größeren Umgebungen.

### 8.1.3 Log-Management

* **ELK-Stack** (Elasticsearch, Logstash, Kibana) oder **Graylog**:

  * Sammeln Proxmox Syslog, Kernel- und Auth-Logs an einer zentralen Stelle.
  * Erlaubt nachträgliche Forensik, Filterung, Dashboards.

***

## 8.2 Wartungsschritte

1. **Updates**

   * `apt-get update && apt-get dist-upgrade -y`, Kernel-Neustart. In Clustern: erst Node1 updaten, VMs migrieren, Node2 updaten usw.

2. **Storage-Checks**

   * ZFS: `zpool scrub rpool` (z.B. monatlich).
   * LVM: `pvs, vgs, lvs` inspizieren.
   * SMART-Werte festplatten: `smartctl -A /dev/sdX`.

3. **Backups checken**

   * Logs der Backup-Jobs lesen (_Datacenter → Backup_).
   * Zeitgesteuerte Test-Restores durchführen.

4. **HA-Funktionstests**

   * Geplante Node-Wartung: Node offline nehmen (Maintenance Mode) → Sehen, ob VMs korrekt migrieren/neu starten.

5. **Cleanup**

   * Alte, nicht mehr genutzte ISOs/Backups löschen.
   * Verwaiste VMIDs oder Container-IDs aufräumen.

***

# **Kapitel 9 – Automatisierung**

In Test- und Produktionsumgebungen mit vielen VMs lohnt es sich, repetitive Tasks zu **automatisieren**.

## 9.1 Cron-Jobs und Scripts

1. **Backup-Skripte**: Selbstdefinierte VZDumps, rsync oder rclone, die man per Cron anstößt.
2. **ZFS-Scrub**: `zpool scrub <pool>` wöchentlich.
3. **Logrotation**: Debian standard, aber man kann z.B. Syslog via Cron an externe Tools pipen.

## 9.2 Ansible

* **Agentenlos**: Man spricht per SSH die Proxmox-Hosts an.

* **Proxmox-Module**: In `ansible-galaxy collection install community.general` sind proxmox\_kvm, proxmox\_storage etc.

* **Use Cases**:

  * VMs anlegen, löschen, Snapshot & Backup, Cluster-Updates.
  * Im großen Stil identische Umgebungen deployen.

## 9.3 Hooks und die Proxmox REST API

* **Hooks**: Per VMID-hookscript kann man Shell-Skripte triggern, wenn eine VM startet/stoppt/migriert. Praktisch für benutzerdefinierte Meldungen oder Post-Provisioning.
* **REST API**: Mit Tools wie `pvesh` (lokal) oder HTTP(S) + Token Authentication kann man sämtliche Proxmox-Kommandos programmatisch ausführen.
* Integration in CI/CD-Pipelines: Bei bestimmten Releases kann man VMs anlegen, Tests fahren, VMs archivieren.

***

# **Kapitel 10 – Fazit**

Mit diesem letzten, umfangreichen Teil haben wir das Fundament für eine **komplette** Proxmox VE Test- oder auch Produktionsumgebung gelegt:

1. **Clustering und HA**: Dank Corosync und HA-Manager können Sie auf mehreren Knoten VMs/Container hochfahren, ausfallsicher betreiben und bei Hardwareproblemen automatisch Failover durchführen.
2. **Backup & Wiederherstellung**: VZDump, ZFS-Send/Receive oder der Proxmox Backup Server sorgen für schnelle und konsistente Datensicherungen.
3. **Monitoring & Wartung**: Tools wie Prometheus/Grafana, Zabbix oder Log-Management-Systeme helfen, Ressourcen und Logs im Blick zu behalten. Regelmäßige Updates, Storage-Checks und Backup-Tests halten das System stabil.
4. **Automatisierung**: Durch Ansible, Hooks oder die REST API wird das Verwalten Dutzender VMs und Container zum Kinderspiel.
5. **Sicherheit**: **SSH-Härtung**, 2FA, CrowdSec, RBAC und Keycloak-SSO mindern die Angriffsfläche erheblich und schützen Ihre Virtualisierungslandschaft vor unberechtigten Zugriffen.

## 10.1 Abschließende Tipps

* **Klein anfangen**: Zuerst 1–2 Nodes, ZFS oder EXT4, VLANs, schrittweise mehr.
* **Dokumentation pflegen**: Jede Änderung (neue VLAN IDs, Firewall-Regeln) sauber notieren, sonst verliert man bei Störungen den Überblick.
* **Testen, testen, testen**: Ob HA-Failover, Snapshots, Backup-Restore – probieren Sie alles in einer ruhigen Minute aus.
* **Community & Support**: Die Proxmox-Community ist sehr aktiv. Bei tieferen Problemen mit QDevices, HA oder Keycloak-SSO lohnt sich ggf. auch ein **Enterprise-Support-Vertrag**.

## 10.2 Weiterführende Ressourcen

1. **Offizielle Proxmox-Doku**: <https://pve.proxmox.com/pve-docs/>
2. **Proxmox Backup Server**: <https://pbs.proxmox.com/docs/>
3. **CrowdSec**: <https://doc.crowdsec.net/>
4. **ZFS on Linux**: <https://openzfs.github.io/openzfs-docs/>
5. **Keycloak**: <https://www.keycloak.org/documentation>
6. **pfSense**: <https://www.pfsense.org/docs> / OPNsense: <https://docs.opnsense.org/>

***

**Vielen Dank für die Lektüre dieser erweiterten, fachbuchartigen Proxmox VE Dokumentation.**\
Wir hoffen, dass Sie damit nicht nur eine solide Testumgebung aufbauen können, sondern auch Inspiration für produktive, hochverfügbare und sichere Virtualisierungslösungen finden. Sollten Sie Fragen oder Anmerkungen haben, steht die Proxmox-Community sowie ggf. der offizielle Proxmox-Support stets zur Verfügung.
