---
title: 'Administrator-Passwort auf einem Windows Dedicated Server ändern'
slug: "windows-admin-passwort-aendern"
excerpt: 'Hier erfahren Sie, wie Sie das Administrator-Passwort auf einem Windows Dedicated Server ändern.'
section: 'Server Management'
order: 2
---

**Letzte Aktualisierung am 16\. Dezember 2020**

> [!primary]
> Diese Übersetzung wurde durch unseren Partner SYSTRAN automatisch erstellt. In manchen Fällen können ungenaue Formulierungen verwendet worden sein, z.B. bei der Beschriftung von Schaltflächen oder technischen Details. Bitte ziehen Sie beim geringsten Zweifel die englische oder französische Fassung der Anleitung zu Rate. Möchten Sie mithelfen, diese Übersetzung zu verbessern? Dann nutzen Sie dazu bitte den Button «Mitmachen» auf dieser Seite.
>

## Ziel

Bei der Installation oder Neuinstallation eines Windows-Betriebssystems wird Ihnen ein Passwort für den Root-Zugriff zugeteilt. Wie in unserer Anleitung "Einen Dedicated Server sichern"erläutert, empfehlen wir Ihnen dringend, [diesen zu](../dedizierten-server-sichern/){.external} ändern. Wenn Sie Ihr Administratorpasswort verloren haben, müssen Sie es im Rescue-Modus zurücksetzen.

**Diese Anleitung begleitet Sie während des gesamten Prozesses zur Änderung des Admin-Passworts Ihres Servers über die für ein Windows-Betriebssystem verfügbaren Rescue-Modus-Konfigurationen.**

## Voraussetzungen

* Sie verfügen über einen [Dedicated Server](https://www.ovhcloud.com/de/bare-metal/){.external}, auf dem Windows installiert ist.
* Sie sind in Ihrem [OVHcloud Kundencenter](https://www.ovh.com/auth/?action=gotomanager){.external} angemeldet.


## In der praktischen Anwendung

Die folgenden Schritte beschreiben den Vorgang zur Änderung des lokalen Admin-Passworts im OVHcloud Rescue-Modus (basierend auf Linux), der jederzeit verfügbar ist. Wenn Sie Windows PE (WinRescue) verwenden möchten, lesen Sie am Ende [dieser Anleitung die entsprechende Vorgehensweise](./#zurucksetzen-des-admin-passworts-mit-winRescue_1). 

### Schritt 1: Server im Rescue-Modus neu starten

Das System muss im Rescue-Modus gestartet werden, bevor das Administratorpasswort geändert werden kann. Loggen Sie sich im [OVHcloud Kundencenter ein](https://www.ovh.com/auth/?action=gotomanager), gehen Sie in den Bereich `Bare Metal Cloud`{.action} und wählen Sie Ihren Server in der linken Navigationsliste unter `Dedicated Server`{.action}.

Der Netboot-Modus wird auf "rescue64-pro (Customer rescue system (Linux))"umgestellt. Suchen Sie "Boot" im Bereich **Allgemeine Informationen** und klicken Sie auf `...`{.action} und dann auf `Ändern`{.action}.
<br>Setzen Sie im angezeigten Fenster im Menü **einen Haken bei Im** Rescue-Modus booten und wählen Sie "rescue64-pro"aus. Geben Sie im letzten Feld eine E-Mail-Adresse an, wenn die Login-Daten an eine andere Adresse als die Hauptadresse Ihres OVHcloud-Accounts gesendet werden sollen. 

Klicken Sie auf `Weiter`{.action} und dann auf `Bestätigen`{.action}.

![Rescuemode](images/adminpw_win_01.png){.thumbnail}

Wenn die Änderung abgeschlossen ist, klicken Sie auf `...`{.action}. rechts von "Status"im Bereich **Status der Dienste**.
<br>Klicken Sie auf `Neu`{.action} starten und der Server wird im Rescue-Modus neu gestartet. Die Durchführung dieser Operation kann einige Minuten dauern.
<br>Sie können den Fortschritt im Tab Tasks `überprüfen`{.action}. Es wird Ihnen eine E-Mail mit den Identitäten (einschließlich des Verbindungspassworts) des Root-Benutzers des Rescue-Modus zugesandt.

![rescuereboot](images/adminpw_win_02.png){.thumbnail}

Weitere Informationen zum Rescue-Modus finden Sie [in dieser Anleitung](../ovh-rescue/).

### Schritt 2: Systempartition mounten

Verbinden Sie sich via SSH mit Ihrem Server. Wenn nötig, lesen Sie die Anleitung.
<br>von Windows Server. Die Partitionen werden als "Microsoft LDM Data"bezeichnet.

```
# fdisk -l
Disk /dev/sda: 1.8 TiB, 2000398934016 Bytes, 3907029168 Sectors
Einheiten: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 54A5B25A-75B9-4355-9185-8CD958DCF32A
 
Device          Start        End    Sectors  Size Type
/dev/sda1        2048     718847     716800  350M EFI System
/dev/sda2      718848     720895       2048    1M Microsoft LDM metadata
/dev/sda3      720896     980991     260096  127M Microsoft reserved
/dev/sda4      980992 3907028991 3906048000  1.8T Microsoft LDM data
/dev/sda5  3907028992 3907029134        143 71.5K Microsoft LDM data
```

In diesem Beispiel ist "sda4"die Systempartition, die durch ihre Größe bestimmt wird. Im Allgemeinen gibt es auch eine zweite Spiegelpartition, die in diesem Fall als "/dev/sdbX" **bezeichnet** wird. In den meisten Fällen verfügt der Server über mehrere Festplatten mit identischen Partitionsschemata. Für den Vorgang zur Zurücksetzung des Passworts ist nur das erste Wort wichtig. 

Wählen Sie diese Partition aus:

```
# mount /dev/sda4 /mnt
```

Überprüfen Sie den Mountpunkt:

```
# lsblk
NAME MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
sdb 8:16 0 1.8T 0 disk
├ sdb4 8:20 0 1.8T 0 part
├ ─ sdb2 8:18 0 1M 0 part
├ ─ sdb5 8:21 0 71 5K 0 part
├ ─ sdb3 8:19 0 127M 0 part
└─ sdb1 8:17 0 350M 0 part
sda 8:0 0 1.8T 0 disk
├─sda4 8:4 0 1.8T 0 part /mnt
├─sda2 8:2 0 1M 0 part
├ ─ sda5 8:5 0 71 5K 0 part
├─sda3 8:3 0 127M 0 part
└─ ─ sda1 8:1 0 350M 0 part
```

Im oben stehenden Beispiel ist die Operation erfolgreich. Wenn das Mounten fehlgeschlagen ist, erhalten Sie wahrscheinlich eine ähnliche Fehlermeldung: 

```
The disk contains an unclean file system (0, 0).
Metadata kept in Windows cache, refused to mount.
Failed to mount '/dev/sda4': Operation not permitted
The NTFS partition is in unsafe state. Please resume and shutdown
Windows fully, or mount the volume
read-only with the 'ro' mount option.
```

Verwenden Sie in diesem Fall den folgenden Befehl und versuchen Sie dann, die Partition erneut zu mounten.

```
# ntfsfix /dev/sda4
# mount /dev/sda4 /mnt
```

### Schritt 3: Derzeitiges Passwort löschen

In diesem Schritt wird die *SAM*-Datei mit einem Tool zur Löschung des Passworts des Admin-Benutzers bearbeitet. Greifen Sie auf den richtigen Ordner zu und registrieren Sie die Windows-Benutzer:

```
# cd /mnt/Windows/System32/config
/mnt/Windows/System32/config# chntpw -l SAM

chntpw version 1.00 140201, (c) Petter N Hagen
Hive <SAM> name (from header): <\SystemRoot\System32\Config\SAM>
ROOT KEY at offset: 0x001020 * Subkey indexing type is: 686c <lh>
File size 65536 [10000] bytes, containing 8 Seiten (+ 1 Headerpage)
Used for data: 359/39024 BLÖCKE/BYTES, unused 33/18064 Blöcke/Bytes.

| RID -|---------- Username ------------| Admin? |- Lock? --|
| 03e8 | admin                          | ADMIN  | dis/lock |
| 01f4 | Administrator                  | ADMIN  | dis/lock |
| 01f7 | DefaultAccount                 |        | dis/lock |
| 01f5 | Guest                          |        | dis/lock |
| 01f8 | WDAGUtilityAccount             |        | dis/lock |
```

Löschen Sie das Passwort des Admin-Benutzers mit folgendem Befehl: (Wählen Sie "Administrator"aus, wenn "Admin"nicht existiert.)

```
# chntpw -u-admin SAM
chntpw version 1.00 140201, (c) Petter N Hagen
Hive <SAM> name (from header): <\SystemRoot\System32\Config\SAM>
ROOT KEY at offset: 0x001020 * Subkey indexing type is: 686c <lh>
File size 65536 [10000] bytes, containing 8 Seiten (+ 1 Headerpage)
Used for data: 361/39344 BLÖCKE/BYTES, unused 35/13648 Blöcke/Bytes.
 
================= USER EDIT ====================
 
RID: 1000 [03e8]a
Username: admin
fullname:
wie:
homedir:
 
00000221 = Users (which has 3 members)
00000220 = Administratoren (which has 2 members)
 
Account bits: 0x0010 =
[ ] Disabled        | [ ] Homedir req.    | [ ] Passwd not req. |
[ ] Temp. duplicate | [X] Normal account  | [ ] NMS account     |
[ ] Domain trust ac | [ ] Wks trust act.  | [ ] srv trust act   |
[ ] Pwd don't expir | [ ] Auto lockout    | [ ] (unknown 0x08)  |
[ ] (unknown 0x10)  | [ ] (unknown 0x20)  | [ ] (unknown 0x40)  |
 
Failed login count: 0, While max triis: 0
Gesamt-Login Count: 5
 
- - - - User Edit Menu:
 1 - Clear (blank) user password
(2 - Unlock and enable user account) [seems unlocked already]
 3 - Promote user (make user an administrator)
 4 - Add user to a group
 5 - Remove user from a group
 q - Quit editing user, back to user select
Select: [q] >
```

Tippen Sie "1"und drücken Sie auf Eingehend ↩. (Verwenden Sie zuerst Option 2, wenn ein "X"gegenüber "Disabled"erscheint.)

```
Select: [q] > 1
Password cleared!
================= USER EDIT ====================
 
RID: 1000 [03e8]
Username: admin
fullname:
wie:
homedir:
 
00000221 = Users (which has 3 members)
00000220 = Administratoren (which has 2 members)
 
Account bits: 0x0010 =
[ ] Disabled        | [ ] Homedir req.    | [ ] Passwd not req. |
[ ] Temp. duplicate | [X] Normal account  | [ ] NMS account     |
[ ] Domain trust ac | [ ] Wks trust act.  | [ ] srv trust act   |
[ ] Pwd don't expir | [ ] Auto lockout    | [ ] (unknown 0x08)  |
[ ] (unknown 0x10)  | [ ] (unknown 0x20)  | [ ] (unknown 0x40)  |
 
Failed login count: 0, While max triis: 0
Gesamt-Login Count: 5
** NT MD4 Hash found. This user probably has a BLANK password!
** LANMAN hash found either. Try Login with no password!
 
- - - - User Edit Menu:
 1 - Clear (blank) user password
(2 - Unlock and enable user account) [seems unlocked already]
 3 - Promote user (make user an administrator)
 4 - Add user to a group
 5 - Remove user from a group
 q - Quit editing user, back to user select
Select: [q] >
```

Tippen Sie "q" ein und drücken Sie auf "Eingehend", um das Tool zu verlassen. Geben Sie "y" ein, wenn Sie eingeladen sind, und drücken Sie auf Eingehend.

```
Select: [q] > q
 
Hives that have change:
 #  Name
 0  <SAM>
Write hive files? (y/n) [n]: ,
 0  <SAM> - OK
```

### Schritt 4: Server neu starten 

Beginnen Sie damit, den Netboot-Modus **im OVHcloud Kundencenter durch Booter** auf der Festplatte [zu](https://www.ovh.com/auth/?action=gotomanager) ersetzen (siehe [Schritt](./#schritt-1-server-im-rescue-modus-neu-starten_1)). 

Teilen Sie die Partition wieder ab und starten Sie den Server mit folgenden Befehlen neu:

```
# cd
# umount /mnt
# reboot

Broadcast message from root@rescue.ovh.net on pts/0 (Wed 2020-05-27 11:28:53 CEST):

System is going down for reboot NOW!
```

### Schritt 5: ein neues Passwort festlegen (IPMI)

Gehen Sie [im OVHcloud Kundencenter](https://www.ovh.com/auth/?action=gotomanager) auf den Tab `IPMI`{.action}, um eine KVM-Session zu eröffnen.

![IPMI](images/adminpw_win_03.png){.thumbnail}

#### Schritt 5.1: für eine neuere Windows-Version

Das Verbindungsinterface sollte eine Nachricht anzeigen, die den Ablauf des Passworts anzeigt.

![pwreset](images/adminpw_win_04.png){.thumbnail}

Das neue Passwort des Admin-Benutzers muss nun zweimal eingegeben werden. Das Bestätigungsfeld ist jedoch noch nicht sichtbar, was bedeutet, dass Sie das erste Feld leer lassen, Ihr neues Passwort in das zweite Feld eingeben und dann die Tabulationstaste ("" ↹") der Tastatur (virtuell) verwenden müssen, um zum dritten Feld ("Passwort bestätigen") zu wechseln.
<br>Geben Sie das Passwort erneut ein und klicken Sie auf den Pfeil, um es zu speichern.

![Enterpw](images/adminpw_win_05.png){.thumbnail}

Klicken Sie auf `OK`{.action} und Sie werden eingeloggt.

![adminlogin](images/adminpw_win_06.png){.thumbnail}

#### Schritt 5.2: für eine ältere Windows-Version

Ein Fenster für die Kommandozeile (cmd) muss sich öffnen, wenn die KVM-Sitzung abgeschlossen ist.

Legen Sie das Passwort des aktuellen Benutzers ("Administrator") fest:

```
net user administrator*
```


![administratorpw](images/adminpw_win_07.png){.thumbnail}

> [!primary]
>
Es wird empfohlen, bei der Eingabe von Passwörtern in dieses Interface die virtuelle Tastatur zu verwenden.
>


### Zurücksetzen des Admin-Passworts mit WinRescue

#### Schritt 1: Server im Rescue-Modus neu starten

Das System muss im Rescue-Modus gestartet werden, bevor das Administratorpasswort geändert werden kann. Loggen Sie sich im [OVHcloud Kundencenter ein](https://www.ovh.com/auth/?action=gotomanager), gehen Sie in den Bereich `Bare Metal Cloud`{.action} und wählen Sie Ihren Server in der linken Navigationsliste unter `Dedicated Server`{.action}.

Der Netboot-Modus wird auf "WinRescue (Rescue System for Windows)"umgestellt. Suchen Sie "Boot" im Bereich **Allgemeine Informationen** und klicken Sie auf `...`{.action} und dann auf `Ändern`{.action}.
<br>Setzen Sie im angezeigten Fenster im **Rescue-Modus einen Haken** und wählen Sie im Menü "WinRescue"aus. Geben Sie im letzten Feld eine E-Mail-Adresse an, wenn die Login-Daten an eine andere Adresse als die Hauptadresse Ihres OVHcloud-Accounts gesendet werden sollen. 

Klicken Sie auf `Weiter`{.action} und dann auf `Bestätigen`{.action}.

![Winrescuemode](images/adminpw_win_08.png){.thumbnail}

Wenn die Änderung abgeschlossen ist, klicken Sie auf `...`{.action}. rechts von "Status"im **Bereich "Status der Dienste**.
<br>Klicken Sie auf `Neu`{.action} starten und der Server wird im Rescue-Modus neu gestartet. Die Durchführung dieser Operation kann einige Minuten dauern.
<br>Sie können den Fortschritt im Tab Tasks `überprüfen`{.action}.
<br>Es wird Ihnen eine E-Mail mit den Zugangsdaten (einschließlich des Verbindungspassworts) des Root-Benutzers des Rescue-Modus zugesandt.

![rescuereboot](images/adminpw_win_02.png){.thumbnail}

Weitere Informationen zum Rescue-Modus finden Sie [in dieser Anleitung](../ovh-rescue/).

#### Schritt 2: Derzeitiges Passwort löschen

Gehen Sie [im OVHcloud Kundencenter](https://www.ovh.com/auth/?action=gotomanager) auf den Tab IPMI` `{.action}, um eine KVM-Session zu eröffnen.

![IPMI](images/adminpw_win_03.png){.thumbnail}

Um die Passwörter zurückzusetzen, ist das NTPWEdit-Tool erforderlich. Wenn Sie über das KVM eingeloggt sind, öffnen Sie den Browser und laden Sie ihn von der offiziellen [Website herunter](http://www.cdslow.org.ru/en/ntpwedit/). 

Navigieren Sie zum Ordner, in dem sich die heruntergeladene ZIP-Datei befindet, und extrahieren Sie den Inhalt. Öffnen Sie anschließend die ausführbare *ntwedit64*, um die Anwendung zu starten.

![ntpwedit](images/adminpw_win_09.png){.thumbnail}

In diesem Interface können Sie die *SAM*-Datei bearbeiten, um das Passwort des Admin-Benutzers zu löschen. Der Standard-Zugriffspfad des *WINDOWS*-Verzeichnisses ist vorausgefüllt. Öffnen Sie die Datei, um die Liste der Benutzer anzuzeigen, indem Sie auf `Öffnen klicken`{.action}.

Der betroffene Benutzer ist entweder "admin"oder "Administrator" entsprechend der Windows-Version. Sind beide vorhanden, wählen Sie "admin"aus. Klicken Sie anschließend auf `Passwort ändern`{.action}.

![ntpwedit](images/adminpw_win_10.png){.thumbnail}

Lassen Sie im angezeigten Fenster die Felder leer und klicken Sie auf `OK`{.action}. Klicken Sie zum Abschluss auf `Änderungen speichern`{.action} und dann auf `Verlassen`{.action}.

Der Server muss dann neu gestartet werden.

#### Schritt 3: Server neu starten 

Beginnen Sie damit, den Netboot-Modus **im OVHcloud Kundencenter durch Booter** auf der Festplatte [zu](https://www.ovh.com/auth/?action=gotomanager) ersetzen (siehe [Schritt](./#etape-1-redemarrer-le-serveur-en-mode-rescue_1)).

Wählen Sie im KVM-Fenster die Option Neu starten `über`{.action} den Windows-Button "Starten" unten links aus.

Lesen Sie diese Anleitung in Schritt [5: ein neues Passwort (IPMI)](./#etape-5-definir-un-nouveau-mot-de-passe-ipmi) festlegen


## Weiterführende Informationen

[Rescue-Modus](../ovh-rescue/{.external}

[Dedicated Server](../verwendung-ipmi-dedicated-server/){.external}

Für den Austausch mit unserer User Community gehen Sie auf <https://community.ovh.com/en/>.
