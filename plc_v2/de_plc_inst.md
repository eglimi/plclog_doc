# Bedienungsanleitung

Der wisol PLC Logserver besteht aus 2 Teilen: Einem Server, der auf einem PC läuft und auf Nachrichten einer SPS wartet, diese entgegennimmt und in eine Datenbank schreibt. Auf diesen Server können Sie mittels Webbrowser zugreifen und sich die Einträge anzeigen lassen.
Auf der anderen Seite ist die SPS Komponente, welche ein Interface (FC) zur Verfügung stellt. Sowie Sie über einen Aufruf dieses FC's eine Nachricht versenden, legt dieser FC die Nachricht im Sendepuffer ab.
Der log_master wird zyklisch im OB1 aufgerufen und schickt die Nachrichten im Sendepuffer als UDP Nachricht über das Ethernet an den Server.

## Serverkomponente

Die Serverkomponente ist als Download für die Betriebssysteme:
* Windows 7 (64Bit)
* Windows 7 (32Bit)
* Linux
* Raspberry
erhältlich. Bitte laden Sie die entsprechende Komponente herunter und führen Sie diese auf Ihrem PC (oder Server) aus.
Wenn der Server erfolgreich gestartet ist, sollten folgende Ausgaben ersichtlich sein:
2015/01/25 18:42:38 Preparing database.
2015/01/25 18:42:38 Database prepared.
2015/01/25 18:42:38 Listening for PLC packets on UDP host 0.0.0.0 and port 3810
2015/01/25 18:42:38 Program Running. Waiting for termination.
Webserver running on :8080

Sie können nun die Website mit den Logmeldungen über "http://localhost:8080" oder "http://IP_adresse_des_Servers:8080" erreichen. Wenn Sie den Server zum ersten Mal starten, sind noch keine Einträge in der Liste ersichtlich. Ein Stoppen und erneutes Starten des Servers wird die Logmeldungen in der Datenbank nicht löschen. Die alten, gespeicherten Meldungen sind weiterhin ersichtlich. Wenn Sie die Meldungen löschen möchten, müssen Sie die Datei "plclog.db" entfernen. Beim nächsten Start des Servers wird dann eine neue Datenbank angelegt.

## SPS Komponenten

Ein Beispielprojekt inklusive der Implementierung und aller notwendigen Bausteine finden Sie auf unserer [Website][plclog]. Der Sourcecode ist unter der [MIT Lizenz][mit] freigegeben.

### Hardwarekonfiguration

In der Hardwarkonfiguration muss keine Anpassung vorgenommen werden. Die Verbindungen zu dem Logserver und die Zuordnung der IP-Adresse zum Server erfolgen während der Laufzeit. Dies hat den Vorteil, dass Sie die Komponenten in eine laufende Anlage integrieren können, ohne dass Sie die SPS stoppen müssen, um die Verbindungsparameter einzustellen.

Einzige Voraussetzung ist, dass Sie die Ethernetschnittstelle einem Netzwerk zugeordnet haben und die SPS über eine eigene IP-Adresse, Subnet und gegebenenfalls Default-Gateway (Router) verfügt.

### Vorbereiten des Programmes

Zum Betrieb des PLClogs sind folgende Siemens Standardbausteine notwendig:
* FB65 TCON
* FB66 TDISCON
* FB67 TUSEND
Sie finden diese Bausteine in der Standardlibrary von Siemens unter Communication Blocks. Es empfiehlt sich, die originalbausteine von Siemens zu nehmen, die auf Ihre SPS passen. So können sich die Bausteine zur Kommunikation bei der S7-300 und S7-400 unterscheiden.

Des weiteren benötigen Sie folgende Bausteine von uns:
* FB200 log_master
* FC200 log_interface
* DB200 log_masterInstance (InstanzDB des FB200)
* DB201 log_buffer
* optional für AWL/KOP/FUP Programme den DB3

Selbstverständlich können Sie sämtliche Bausteine umbenennen - auf die Funktion hat dies keinen Einfluss.

Es befindet sich ebenfalls die SCL Source im Projekt. Daraus können Sie sich die Bausteine auch selbst komplett neu generieren. (Bis auf den DB3, welcher nicht in der Source enthalten ist).

### Konfiguration

Nachdem Sie die Komponenten auf der SPS installiert haben, müssen Sie den Empfänger der Nachrichten (den Server) spezifizieren. Dies geschieht im InstanzDB des log_masters (DB200). 
Sie können dies entweder in dem FB200 in der Schnittstellenbeschreibung anpassen (Schnittstelle -> STAT -> dest_ip_settings -> ip -> field_1 bis 4, wobei Feld 1 das erste Byte der Adresse (192), Feld 2 das zweite Byte der Addresse (168) etc. enthält. Stellen Sie auch die localdeviceit ein. (Steckplatz 2 der Ethernetschnittstelle entspricht der Adresse 2).

Alternativ können Sie direkt den InstanzDB des FB200, den DB200 anpassen. Die Addresse ist ab Addresse 2 (dest_ip_settings) abgelegt. Stellen Sie auch hier die localdeviceid ein (Steckplatz 2 der Ethernetschnittstelle entspricht der Adresse 2. [Steckplatz ID][1]

Selbstverständlich können Sie die Anpassungen auch direkt in der SCL Quelle vornehmen.

Sie können die Adresse jederzeit während der Laufzeit durch ändern der Parameter im DB200 anpassen. Die Änderungen werden sofort wirksam, ohne dass die CPU gestoppt werden muss.

### Einbinden in das Programm

Zum Aktivieren des Logservers müssen Sie diesen (FB200) einerseits im OB100 (Anlauf OB) mit dem Flag startup=true, als auch im OB1 (zyklisch) mit dem Flag startup:=false aufrufen. Es ist wichtig, dass der log_master zyklisch aufgerufen wird, da die verwendeten Siemenskomponenten asynchrone Bausteine sind, die nicht in dem sie aufrufenden Zyklus enden.

Der Logserver läuft nun und ist bereit, Nachrichten entgegenzunehmen. 

#### Aufrufe unter SCL:

Unter SCL sieht ein Loggingaufruf folgendermassen aus:

log_interface(machine := 1,   // Machine Number for the Server UI z.B. Auto
              part    := 5,   // Machine Part   for the Server UI z.B. Getriebe
              level   := 1,   // Message severity for the UI Color, the message appears in
              msg     := 'Freie Textnachricht mit der ersten Nummer %d !!!',
              par1    := 41,
              par2    := 0);

Dieser Aufruf würde folgende Nachricht auf dem Server erscheinen lassen:
  Freie Textnachricht mit der ersten Nummer 41 !!!

#### Aufrufe unter AWL:

Ein Loggingeintrag unter AWL kann folgendermassen gemacht werden. Bitte beachten Sie, dass unter AWL keine Strings an Funktionen Übergeben werden können. Dies muss mittels Pointer gemacht werden.

Als erstes definieren wir einen DB, der die zu versendenden Textkonserven enthält. In unserem Beispiel ist dies der DB3, bestehend aus diversen einzelnen Textstrings von der Länge 50 Byte.
Ein Aufruf sieht also folgendermassen aus:
      CALL "log_interface"                FC200
       machine :=5                        
       part    :=1
       level   :=3
       msg     :="log_messages".msg1      P#DB3.DBX0.0
       par1    :=3.1415e+000
       par2    :=41e+000

Die Ausgabe in diesem Falle ist abhängig von dem Inhalt des DB3 "log_messages".

#### Aufrufe unter KOP / FUP

Leider sind wir nicht in der Lage, ein Beispiel unter KOP/FUP anzubieten, da in unserer Firma niemand diese beiden Sprachen beherrscht. Wir wären froh, wenn sich jemand finden würde, der uns diese Beispiele zusendet.

### Format einer Log-Nachricht

Eine Nachricht besteht aus drei numerischen Werten (INT), zwei Gleitkomma-Werten (REAL) (Parameter 1 und 2) und einem 50 Zeichen langem Text (STRING).
Mit den INT-Werten kann eine Nachricht kategorisiert werden. Die folgenden Parameter können gesetzt werden: 
1. Maschine: Beschreibt, um welche Maschine es sich handelt.
2. Teil: Beschreibt genauer, um welchen Teil der Maschine es sich handelt.
3. Log-Level: Beschreibt die Wichtigkeit der Nachricht.
4. Message: Freier Text, in dem bei Verwendung von %d der erste REAL Parameter anstelle dieses Platzhalters ausgegeben wird, bei einem zweiten %d wird der zweite Parameter ausgegeben.
5. par1: erster optionaler REAL parameter
6. par2: zweiter optionaler REAL parameter
 
Zum Beispiel wird ein Aufruf mit par1 := 3 und par := 84 bei mitgeben von folgendem String: "Milchtank Nr. %d ist zu %d gefüllt" im UI dargestellt als: "Milchtank Nr. 3 ist zu 84% gefüllt".

Folgende Darstellung erhalten Sie bei Verwendung der Platzhalter:
%g gibt einen REAL Wert aus
%d gibt einen Integer aus




[1]: http://support.automation.siemens.com/WW/llisapi.dll?func=cslib.csinfo&lang=de&objid=51339682&caller=view
[mit]:    http://opensource.org/licenses/MIT
[plclog]: http://wisol.ch/w/products/plclog/









 
