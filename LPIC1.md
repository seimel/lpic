# Module zur Laufzeit beeinflussung und konfigurieren
- `lsmod`:
 - zeigt den status der Module einens laufenden Kernels an
 - grift auf `/proc/modules` zu und gibt das Ergebnis übersichtlich aus

- `modinfo`:
 - `modinfo [Modul] -[option]` gibt informationen zu einem Modul aus
 - `-a` = nur Autor (author)
 - `-d` = nur Beschreibung (describtion)
 - `-l` = nur Lizenz (licens)
 - `-p` = nur übergebene Parameter (parameters)
 - `-n` = nur Dateinames (name)


- `insmod`:
 - Module in den laufenden Kernel integrieren
 - prüft automatisch auf Abhängigkeiten, löst selbige aber nicht automatisch auf, sondern gibt nur eine Fehlermeldung aus
 - Bsp.: `insmod /lib/modules/2.6.11.../kernel/drivers/usb/storage/usb-storage.ko`


- `rmmod`:
 - entfernt nicht benötigte Module aus dem Arbeitsspeicher
 - keine Pfadangebe erforderlich, da `rmmod` mit `/proc/modules` arbeitet
 - Bsp.: `rmmod usb-storage` entfernt den nicht mehr benötigten USB-Speichertreiber
 - es wird keine Erfolgsmeldung ausgegbe, aber ine Fehlermeldung wenn das Modul noch von einem anderem Programm genutz wird
 - `-v` = Verbose-Mode (verbose)
 - `-f` = erzwingt das Entladen (force)  


- `modprobe`:
 - ist eine Kombination aus `insmod` und `rmmod` in einem einzigen Programm
 - kann Abhängigkeiten zwischen Modulen erknnen und evtl. entstehende Probleme lösen
 - kein Pfad nötig da `uname -r` genutzt wird um das Basisverzeichnis der Module zu finden
 - Bsp.: `modprobe -at net`
  - `-a` = all
  - `-t` = type
 - bei Erfolg wird keine Rückmeldung gegeben, nur eine Fehlermeldung bei Fehschlag
 - `-r` = entfernen von Modulen. Das entfernen mehrerer Module gleichzeitig ist möglich. Sie müssen mit Leerzeichen getrennt angegeben werden.
 - `-l` = zeigt alle ladbaren Module an
 - `-lt fs` = zeigt alle Dateisystemmodule an


- `depmod`:
 - Abhängigkeiten von Modulen werden in *modules.dep* zentral festgehalten
 - ohne Option wird eine neue *modules.dep* erstellt, indem vorher die Abhängigkeiten bei allen vorhandenen Modulen abgefragt werden
 - `-n` = trockenlauf, Ausgabe nach stdout
 - `-A` = Schnelldruchgang; es wird überprüft, ob es überhaubt Module gibt, die neuer sind als die in der aktuellen *modules.dep*

# Modulkonfigurationsdateien

- *modules.dep*: speichert die Abhänigkeiten von Modulen
 - Bsp.: `/lib/modules/2.6.11.../kernel sound/pci/snd-a7-codec.ko:/lib/modules/2.6.11.../kernel/sound/core/snd-pcm.ko` hier hängt das Modul *snd-ac97-codec.ko* von dem Modul *snd-pcm.ko* ab


- */proc/sys/kernel*: zur Laufzeit legt der Kernel seine Informationen im `/proc`-Dateisystem ab
 - Änderungen in diesen Dateien werden nach einem Neustart wieder Verworfen, da das `/proc`-Dateisystem legendlich Informationen einbindet, sie sich im Arbeitsspeicher befinden

# Zum Kernel gehörende Dateien und Verzeichnisse

In einem Verzeichnis unterhalb von `/usr/src` wird ein Verzeichnis angelegt, das die genaue Versionsnummer des Kernels enthält. Zum Beip. `/usr/src/linux-2.6.11..`. Zusätzlich wird ein Softlink auf `/usr/src/linux` erstellt.

Der statische Teil des lauffähigen Kernels befindet sich im Verzeichnis `/boot`. Normalerweise ist ein Softlink *vmlinux* erstellt, welcher auf den tatsächlichen Namen des Kernels zeigt.

In `lib/modules` wird für jeden installierten kernel ein Unterverzeichnis angelegt. Hier befinden sich bis zu Version 2.4, Unterverzeichnisse für verschiedene Kategorien von Modulen. Ab 2.4 gibt es zunächst das Unterverzeichnis *kernel* und darauf folgend die Unterverzeichnisse mit den jeweiligen Kategorien.

Bsp.: `/lib/modules/2.6.11.../kernel/fs` = für Filesystem-Module

Hier befindet sich auch die `/lib/modules/2.6.11.../modules.dep`

Moduldateien selbst sind durch einen Compiler erstellte Dateien mit der Endung **.o** für **O**bjekt. Ab Version 2.6 und höher ist die Endung **.ko** für **K**ernel-**O** bjekt.

# Die Gerätedateien für HDDs und CD-Roms

Sowohl physikalische, als auch logische Laufwerke werden unter Linux als Gerätedaten unterhalb von `/dev` dargestellt. Es handelt sich hierbei nicht um Verzeichnise auf der Festplatte, sondern um eine Präsentation der Geräte durch den Kernel.

Die IDE-Geräte sind:

- `/dev/hda` = primary master
- `/dev/hdb` = primary slave
- `/dev/hdc` = secondary master
- `/dev/hdd` = secondary slave

Die Dateien für SCSI und SATA (unter Ubuntu auch IDE):

- `/dev/sda` = first SCSI/SATA-device
- `/dev/sdb` = second SCSI/SATA-device
usw..

# Die Gerätedateien für Partitionen
Aufgrund des Aufbaus eines **MBR** (**M**atser **B**oot **R** ecord) und der darin enthaltenen Partitionstabelle, können azf einer HDD nur vier Partiotionen erstellt werden. Es wird zusätzlich zwischen primären und erweiterten Partitionen unterschieden. Eine primäre Partition ist direkt ansprechbar, d.h. sie ist formatierbar und benutzbar. Es können vier primäre Partitionen auf einer HDD koexsistieren. Eine erweiterte Partition dient als Behälter für logische Partitionen. In einer logischen Partition ist platz für 60 logische Partitionen (IDE) oder 12 (SATA/SCSI).

Primäre und erweiterte Partitionen werden von 1-4 durch nummeriert. Die erste logische bekommt immer der Ordnungszahl 5 zugewiesen. Eine partitionierung kann z. B. so aussehen:

- `/dev/hda1` = erste Partition auf prim. Master
- `/dev/hda2` = zwiete Partition auf prim. Master
- `/dev/hda3` = einzige erweiterte Partiteion
- `/dev/hda5` = erste logische Partition
- `/dev/hda5` = zweite logische Partition
- `/dev/hda6` = dritte logiscge Partition
 usw..

*Das Selbe gilt für SATA & SCSI, nur dass es keinen primären oder sekundären Master gibt*


# Resourcen für Hardwarekomponenten
Das Verzeichnis `/proc` ist kein Verzeichnis im eigentlichen Sinne, sondern bildet Parameter des Kernels ab. So kann es zur Abfrage des Kernels verwendet werden.

- `/proc/interrupts` = Infos zu verwendeten Interrupts
- `/proc/ioports` = Infos über von Hardware verwendete I/O-Adressen
- `/proc/dma` = ist eine Liste von verwendten DMA-Kanälen
- `/proc/pci` = veraltet! unter neuen Versionen: `/proc/bus/pci`; enthält Informationen zum PCI-Bus

# Der PCI-Bus

**PCI** = **P** eripheral **C** omponent  **I** nterconect
Mir `lspci` können Informationen über den PCI-Bus ausgegeben werden. Das Kommando bezieht sich dabei auf das Verzeichnis `/proc/bus/pci` und stellt die Informationen auf übersichtiliche Art und Weise dar.

- `lspci -v[vv]` = verschieden Verbose-Stufen
- `-t` = stellt PCI-Gräte in einer Baumstruktur dar (tree)


# USB (Universal Serial Bus)
Um 1996 von Intel entwickeltes Bus-System zum anbinden von Peripherie-Geräten an einen PC.


# USB-Host-Controlle-Typen

## USB 1.1
- **OHCI** (**O**pen **H**ost **C**ontroller **I** nterface)
- **UHCI** (**U**niversal **H**ost **C**ontroller **I** nterface)

Unter Linux muss der jeweils passende Treiber geladen werden: `usb-ohci.o` oder `usb-uhci.o`.

## USB 2.0
- **EHCI** (**E**nhanced **H**ost **C**ontroller **I** nterface)
Derpassende Treiber ist logischerweise `usb-ehci.o`


# USB-Klassen

- Eingabegeräte = HID (Human Interface Device): `hid.o`
- USB-Massenspeicher: `usb-storage.o`

Es wird für jede Klasse ein Treiber benötigt. Wenn ein Gerät initialisiert wurde, wird unerhalb von `/proc` ein Verzeichnis angelegt. Da die Dateien in binärer Form hinterlegt sind, sind sie nich mit einem Editior einsehbar. Um Informationen zum USB-Geräten anzuzeigen wir `lsusb` benutzt. `lsusb -t` zeigt eine Baumstruktur an.
Um Inforamationen über ein bestimmtes Gerät zu bekommen: `lsusb - d Vendor:ProduktNR. -v`.
Aus Gründen der Kompatiblität zu `udev` heutzutage nur noch `/dev/bus/usb`.


# USB-Module automatisch laden

## `usbmgr`
- das Gerät übermittelt beim einstecken Vendor ID und Produkt ID
- in der Datei `/etc/usbmgr/usbmgr.conf` findet `usbmgr` anhand der IDs die passenden Treibermodule, lädt diese in den Arbeitsspecher und führt sie aus.
- `/etc/usbmhr/preload.conf` = Treiber die schon beim booten geladen werden sollen
- `/etc/usbmgr/host` = enthält Namen des Treibers für den Host (ohci, uhci oder ehci)

## `hotplug`
- verbreiteter als `usbmgr`, da auch PCMCIA und Firewire bedient werden kann
- wenn ein Gerät angeschlossen wird, sendet der Kernel Informationen über das Gerät an `hotplug`
- `/etc/hotplug` enthält zu den Komponenten passende Skripte, die dann die Installation der Treibermodule übernehmen.

# Coldplug und Hotplug
Hotplug-Geräte könen während des laufenden Betriebs in den Computer eingebaut werden. Für Coldplug-Geräte muss der Computer herrunter gefahren werden und ausgeschaltet werden.

# Das virtuelle Datisystem sysfs
sysfs exportiert, ähnlich `/proc`, Informationen über Treibermodule des Kernels in für Benutzer sichbares Verzeichnis, `/sys` z. B. `/sys/bus` oder `sys/devices`.
Jedes Unterverzeichnis repräsentiert ein Treibermodell des laufenden Kernels. Symbolische Links innerhalb der Verszeichnisse stellen den Zusammenhang zwischen Geräte, z. B. Controller und HDD, dar. Das Verzeichnis `/sys` wird dynamisch generiert, enthält also keine Informationen üder nicht vorhandene Geräte.

# udev, hald & dbus
hald hat die Aufgabe, dbus darüber zu Informieren wenn ein Wechsellaufwerk an das System angeschlossen oder entfernt wird.

udev werwaltet bei modernen Kernel-Versionen das `/dev`-Verzeichnis und sorgt dafür das nur Geräte angezeigt werden, die auch wirklich an den Computer angeschlossen sind.

# Das System starten

## Algemeines
Nach dem Einschschalten des Computers werden zunächst einige Prozesse vom BIOS ausgeführt. U. a. wird der POST (PowerOnSelfTest) ausgeführt, wo die grundsätzliche funktion des Systems überprüft wird. Zusätzlich wird ein Schnelltest des Artbeitsspeicher durchgeführt und seine größe ermittelt. Eine gründliche Hardwarediagnose kann dieser Test allerdings nicht ersetzten. Nach dem der Vorgang abgeschlossen wurde, sucht das BIOS nach einem Betriebssystem oder einem Programm welches ein Betriebssystem starten kann. Die Suchfolge wird im BIOS-Setup festgelegt. Sie kann Disketten-Laufwerke, CD-ROMs, HDDs aber auch USB-Wechseldateträger enthalten. Im weiteren Verlauf wird von einer HDD ausgegangen.
Eine HDD kann in bis zu vier "echte" Partitionen aufgeteilt sein. Die Partitionen werden in einer Partitionstabelle im MBR verwaltet. Der MBR ist genau 512 Byte groß und befindet sich am Anfang der HDD (Sektor0 / Spur 0). Das BIOS liest nun einfach den im MBR enthaltenen Hexcode aus. Sollte ein Bootloader installiert sein, gibt das BIOS nun die Kontrolle an diesen ab. Falls kein Bootloader vorhanden ist, konsultiert das BIOS die Partitionstabelle und sucht nach einer Startfähigen Partition. Spätestens hier sollte sich nun ein Betriebssystem oder ein Bootloader befinden.

## Kernel-Parameter
Beim Start des Systems ist es möglich dem Kernel Parameter mitzugeben. Die Parameter können entweder in der .conf-Datei des Bootloaders oder beim Start von Hand hinterlegt werden. Letzeres macht natürlich nur Sinn wenn dem Kernel ausnahmsweise, zu Testzwecken o. Ä. Parameter übergeben werden müssen. Wenn Parameter bei jedem Systemsstart übergeben werden sollen, müssen diese in conf-Datei des Bootloaders hinterlegt werden.
Es ist z. B. möglich den init-Prozess gegen die /bin/bash auszutauschen um z. B. ein vergessenes root-Passwort zu ändern.

## Startprotokollierung
