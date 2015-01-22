# plclog Bedienungsanleitung

Der wisol PLC Logserver besteht aus 2 Teilen: Zum einen der Server, welcher Nachrichten einer SIEMENS SPS entgegennimmt und verarbeitet. Zum anderen die SPS, welche eben diese Nachrichten versendet. Diese Bedienungsanleitung enthält die nötigen Informationen, um eine SPS einzurichten und die Nachrichten zu einem wisol Logserver senden zu können.

Zudem bietet dieses Dokument auch einen Einblick in das S7-Programm des PLC Logservers. Im Appendix finden Sie eine Auflistung aller im Programm verwendeten Bausteine und Funktionen. 

Die Anleitung richtet sich an Personen, welche den PLC Logserver in ein bestehendes System einbinden, oder unser S7-Programm als Vorlage für ein neues Projekt verwenden möchten. Das Programm wurde grösstenteils in der S7 Hochsprache SCL (structured control language) geschrieben. Sämtliche SCL-Quellen werden mitgeliefert und können beliebig angepasst werden. Falls Sie keine S7-SCL Lizenz besitzen, können Sie die in AWL übersetzten Bausteine gleichermassen in Ihrem Programm verwenden. 

Diese Anleitung setzt Grundkenntnisse über die Programmierung einer Speicherprogrammierbaren Steuerung von SIEMENS unter Anwendung des SIMATIC Managers (STEP7 V5.5) voraus. 

Unser S7-Programm finden Sie auf unserer [Webseite][plclog]. Der S7 Source Code ist unter der [MIT Lizenz][mit] freigegeben.

## Hardwarekonfiguration 

Falls Sie den PLC Logserver in ein bestehendes System einbinden möchten, können Sie die Schritte bis zur Einrichtung der Verbindungsparameter ignorieren.

Nachdem Sie unser vorbereitetes S7-Programm [heruntergeladen][plclog] und entpackt haben, öffnen Sie dieses mit dem SIMATIC Manager. 
Als erstes muss die Hardwarekonfiguration unseres Programmes Ihren Gegebenheiten anpasst werden. Ersetzen Sie die in unserem Programm projektierte CPU (CPU317-2PN/DP V2.6) mit Ihrer eigenen und fügen Sie die Module (zum Beispiel ein CP-Modul, falls Ihre CPU über keine PROFINET-Schnittstelle verfügt) die Sie sonst noch verwenden möchten hinzu.

### Ändern der lokalen IP

Um eine UDP-Verbindung zwischen der SPS und dem PLC Logserver herstellen zu können, müssen sich beide IP-Adressen im selben Adressbereich befinden. 
Um nun die IP-Adresse der SPS oder des Kommunikationsmoduls ändern zu können, müssen Sie erneut im SIMATIC Manager die Hardwarekonfiguration aufrufen. Jetzt öffnen Sie mit der rechten Maustaste die Objekteigenschaften der PROFINET-Schnittstelle, und wählen die Schaltfläche "Eigenschaften". Hier können Sie in den Feldern "IP-Adresse" und "Subnetzmaske" Ihre gewünschten Adressen eingeben.

![Hardwarekonfiguration][hw]

## Einrichten der Verbindungsparameter

Als nächstes muss im S7-Programm die IP-Adresse des PLC Logservers und die Steckplatz ID der PROFINET-Schnittstelle zugewiesen werden. Über was für eine [Steckplatz ID][1] die PROFINET-Schnittstelle verfügt ist wichtig für die SIEMENS eigenen Funktionsbausteine. Diese werden  in unserem Programm verwendet um eine Netzwerkverbindung aufzubauen. 
In der Hardwarekonfiguration sehen Sie über was für eine Steckplatz ID Ihre SPS oder CP-Modul verfügt (siehe Bild oben: Steckplatz X2 = Steckplatz 2).

Um das einstellen der IP-Adresse des PLC Logservers und der Steckplatz ID zu vereinfachen, haben wir den Datenbaustein DB109 erstellt. Dieser ermöglicht es beide Einstellungen am selben Ort zu ändern. Sie können diese Änderungen entweder mit dem KOP/AWL/FUP-Editor oder mit dem SCL-Editor durchführen. Für den SCL-Editor benötigen Sie jedoch eine S7-SCL Lizenz.

Bitte beachten Sie, dass der Datenbaustein neu [initialisiert][2] werden sobald dieser angepasst oder verändert wurde. Dies ist unabhängig davon, ob das Programm bereits in die SPS geladen wurde.

### Initialisieren eines Datenbausteins

Im SIMATIC Manager öffnen Sie den entsprechenden Datenbaustein mit dem KOP/AWL/FUP-Editor. Als nächstes wechseln Sie mit der Tastenkombination "Ctrl + 4" von der Deklarationsansicht in die Datenansicht. Hier können Sie über das Dropdown-Menu "Ansicht" die Schaltfläche "Datenbaustein initialisieren" betätigen. Nun werden die aktuellen Werte des Offline DB's mit den Initialwerten überschrieben. Zuletzt muss der initialisierte DB noch mit der Tastenkombination "Ctrl + L" in die SPS geladen werden.

![Initialisieren des Datenbausteins][dbinit]

### Mit dem SCL-Editor

Falls Sie über eine S7-SCL Lizenz verfügen können Sie im SIMATIC Manager unter S7-Programm und dann unter Quellen die SCL-Quelle "ip_configuration" mit dem SCL-Editor öffnen. In der Codezeile 24 bis 27 kann die gewünschte IP-Adresse, und in Zeile 30 die Steckplatz ID eingegeben werden.

Alternativ können Sie die Einstellungen auch im KOP/AWL/FUP-Editor vornehmen. Dies wird im nächsten Abschnitt beschrieben.

~~~ 
1 //******************************************************************
2 //************** Datablock definition for IP-Settings **************
3 //******************************************************************
4 data_block dest_ip_settings
5 title   = 'destination ip settings'
6 family  : 'logutil'
7 author  : 'wisol'
8
9
10 struct
11    localdeviceid :byte;
12 
13    ip: struct
14        field_1 : int;
15        field_2 : int;
16        field_3 : int;
17        field_4 : int;
18 
19    end_struct;
20 end_struct
21 begin
22 
23 (*ENTER THE DESTINATION IP ADDRESS: DEFAULT = 192.168.251.55*) 
24    ip.field_1 := 192; 
25    ip.field_2 := 168; 
26    ip.field_3 := 251;
27    ip.field_4 := 55;
28 
29 (*ENTER THE LOCAL DEVICE ID: DEFAULT = B#16#2*)
30    localdeviceid := B#16#2;
31 
32 end_data_block
~~~


### Mit dem KOP/AWL/FUP-Editor

Im SIMATIC Manager öffnen Sie den DB109 "dest_ip_settings" mit dem KOP/AWL/FUP-Editor. Mit der Tastenkombination "Ctrl + 5" wechseln Sie in die Deklarationsansicht des DB's, wo nun die IP-Adresse bei den Variablen "dest_ip_settings.ip_field1" bis ".ip_field_4" (DB109.DBW2 bis DBW8) eingetragen werden kann. Die Steckplatz ID kann bei der Variable "dest_ip_settings.localdeviceid" (DB109.DBB0) eingegeben werden.

![Ändern der IP/Steckplatz ID im KOP/AWL/FUP-Editor][ipdb]

### Temporäre Änderung

Wenn Sie die IP oder die Steckplatz ID zu Testzwecken ändern möchten, kann dies mit Hilfe einer Variablentabelle erreicht werden. Dabei muss der Datenbaustein DB109 nicht neu initialisiert werden. In unserem S7-Programm finden Sie eine Variablentabelle die bereits die wichtigsten Variablen enthält. Öffnen Sie die Tabelle "testing" die Sie im SIMATIC Manager unter S7-Programm/Bausteine finden. Nun können Sie für die IP-Adresse die Variablen "dest_ip_settings.ip_field_1" bis "ip_field_4" mit einem dezimalen Steuerwert überschreiben. Für die Steckplatz ID überschreiben Sie den hexadezimalen Statuswert der Variable "dest_ip_settings.localdeviceid" mit dem entsprechendem Wert. 

Bitte beachten Sie, dass die SPS die temporären Änderungen verliert sobald ein Neustart erfolgt. Das heisst, bei einem Neustart werden die Werte mit den Initialwerten überschrieben.

![Anpassung der Einstellungen über die Variablentabelle][ipvat]

## Definieren der Log-Nachrichten

Nachdem die Verbindung zwischen der SPS und dem Logserver parametriert ist, können nun Nachrichten definiert und versendet werden. Die nächsten Abschnitte beschreiben, wie und wo eine Nachricht definiert werden kann, und wie diese versendet wird.

### Format einer Log-Nachricht

Eine Nachricht besteht aus drei numerischen Werten (INT), zwei Gleitkomma-Werten (REAL) (Parameter 1 und 2) und einem 50 Zeichen langem Text (STRING). 

Mit den INT-Werten kann eine Nachricht kategorisiert werden. Die folgenden Parameter können gesetzt werden, wobei die Reihenfolge entscheidend ist.

1. Maschine: Beschreibt, um welche Maschine es sich handelt.
2. Teil: Beschreibt genauer, um welchen Teil der Maschine es sich handelt.
3. Log-Level: Beschreibt die Wichtigkeit der Nachricht.

Die Vergabe der Werte ist frei und kann an die Gegebenheiten angepasst werden. Anhand dieser Werte können später Nachrichten wieder gefunden werden.

Die REAL-Werte ermöglichen, 2 beliebige Prozessdaten zu senden. So können z.B. aktuelle Temperaturen, Füllstände, Zeiten, etc. übergeben werden. Diese Werte werden im PLC Logserver in die Nachricht eingebaut.

Mit dem Text wird die eigentliche Nachricht definiert. Diese besteht aus einem Text, welcher maximal 50 Zeichen lang ist (inklusive Leerzeichen). Zudem können mit der Formatanweisungen "%g" die beiden REAL-Werte in die Nachricht eingebettet werden.

Beispiel einer Definition einer Nachricht mit 2 REAL-Werten.

~~~
Temp. Pt100: Mantel = %g; Innen = %g
~~~

Diese Nachricht wird dann vom PLC Logserver zu der folgenden Nachricht zusammengesetzt (angenommen die Werte werden entsprechen eingesetzt):

~~~
M:2 P:1 L:2        Temp. Pt100: Mantel = 47.717; Innen = 68.1789
~~~

### Definieren eines Text-Strings

Die Text-Strings werden im DB3 generiert. Um diese zu bearbeiten, öffnen Sie im SIMATIC Manager mit dem KOP/AWL/FUP-Editor den DB3 "log_messages". Mit der Tastenkombination "Ctrl + 5" wechseln Sie in die Deklarationsansicht des DB's, wo Sie nun die Strings "log_messages.msg1" bis "log_messages.msg99" (DB3.DBW0 bis DBW5096) bearbeiten können. Auch hier müssen Sie anschliessend wieder den DB initialisieren und in die SPS hochladen.

Nachdem Sie die Nachrichten definiert haben, müssen Sie noch die restlichen Werte (INT- und REAL-Werte) der Nachricht zuweisen. Diese Zuweisung erfolgt beim Aufruf der Funktion, welche für das Versenden der Nachrichten zuständig ist.

![Datenbaustein log_messages][db3]

## Funktionsaufrufe

Als nächstes müssen notwendigen Funktionen für die Ausführung des Programmes in den entsprechenden Organisationsbausteine aufgerufen werden. Die Organisationsbausteine wurden in AWL programmiert und sind mit dem KOP/AWL/FUP-Editor öffenbar.

### OB1

In diesem Abschnitt wird der Inhalt des OB1 beschrieben.

~~~
OB1:  "Main Program Sweep (Cycle)"

Netzwerk 1: 
The function FC100 "log_master" is called to initialize the connection between 
the plc and the remote ip address. The return value tells you in which state 
the connection is in.
Calling the function FC103 "log_interface" sends the message like you formatted 
it down below. 

 1 //   CALL  "log_master"			FC100
   //     RET_VAL:="stateLog_master"		MW20

 2 //   U     M     10.6
   //   UN    M      2.0
   //   =     "TestDelay"			M2.1	
   //   U     M     10.6
   //   =     M      2.0

 3 //   U     "TestFlag"			M1.0
   //	U     "TestDelay"			M2.1	
   //   SPBN  M001

 4 //   CALL  "log_interface"			FC103
   //    machine:=2
   //    part   :=1
   //    level  :=2
   //    msg    :="log_messages".msg1		P#DB3.DBX0.0	-- Beispiel Text-String
   //    par1   :=4.771700e+001
   //    par2   :="temp_sensor".pt100_innen	DB444.DBD0

 5 // M001: NOP   0
~~~

Erklärung OB1:

1. Bei jedem Zyklus wird der FC100 aufgerufen. Im "log_master" wird der Sendepuffer auf versendbare Log-Nachrichten durchsucht. Zusätzlich wird im "log_master" die Funktion "log_reinit" (FC101) aufgerufen, welche für das Aufbauen und Beenden der Netzwerkverbindung zuständig ist.

2. Mit Hilfe des Taktmerkerbytes (MB10) und eines zweiten Merkers entsteht eine Zeitverzögerung. Diese verzögert das Senden der Nachrichten. Ansonsten würde bei jedem Programmzyklus eine Log-Nachricht versendet werden (sofern M1.0 gesetzt und die Verbindung initialisiert ist).

3. Wenn der Merker "TestFlag" nicht gesetzt ist, überspringt das Programm den Aufruf von FC103 und springt zu Punkt 5. 

4. Wenn "TestFlag" gesetzt ist, wird die Funktion "log_interface" aufgerufen. Im FC103 wird die Log-Nachricht in den Sendepuffer geschrieben und an den PLC Logserver versendet. Bei diesem Aufruf werden auch die oben erwähnten restlichen INT- und REAL-Werte der Nachricht festgelegt.

5. Sprungmarke von SPBN1 und Nulloperation

Die Punkte 1., 3., 4. und 5. müssen zwingend implementiert sein um die Funktionalität zu gewährleisten. Die Bedingungen in Schritt 2. unter welchen die Funktionen aufgerufen werden, können beliebig angepasst werden. Beachten Sie, dass der FC100 zwingend zyklisch aufgerufen werden muss.

Beschreibung der Rückgabewerte (MW20 "stateLog_master") von FC100:

- 0: Normalzustand: Verbindung wurde aufgebaut und die SPS ist bereit Nachrichten zu versenden.

- 1: Verbindung Fehlerhaft: Die SPS kann den anderen Ethernet-Teilnehmer (PLC Logserver) nicht erreichen. Vermutlich befinden sich die beiden IP-Adressen nicht im gleichen Adressbereich oder es besteht keine physische Verbindung.

- 2: Zurücksetzen der Verbindung fehlgeschlagen: Die Funktion "log_reinit" (FC101) wurde nicht richtig ausgeführt. Entweder die SPS neu starten oder die drei Variablen "log_data.init_done", "log_data.disc_done" und "log_data.init_start" mit Hilfe der Variablentabelle mit dem Wert 0 überschreiben

- 3: Sendepuffer overflow: Entweder werden Nachrichten schneller in den Puffer geschrieben als versendet werden können oder es wird zwar in den Puffer geschrieben aber nichts versendet. 


### OB100
Der OB100 und dessen Funktionsaufruf ist zwingend zu implementieren. Auf Grund der Eigenheiten der SIEMENS Funktionsbausteine müssen die Bedingungen unter welchen diese in FC101 aufgerufen werden mit dem FC102 zurückgesetzt werden. Wenn das nicht gemacht wird, kann bei einem Neustart der SPS die Verbindung nicht korrekt beendet und aufgebaut werden.

~~~
OB100:  "Complete Restart"

Netzwerk 1: 
This Program Block is needed to reset the connection state when the PLC is 
starting up.

        UC    "log_startup"			FC102
~~~



## Appendix A: Baustein-Übersicht

Eine Übersicht über alle der im S7-Programm verwendete Bausteine und Funktionen.

![Datenbausteine][dblist]

### Organisationsbausteine

OB1 - Standard OB, ruft alle nötigen Funktionen auf

OB100 - Anlauf OB, setzt bei einem Neustart der SPS die Netzwerkfunktionen zurück

### Funktionen

FC100 - Ruft FC101 auf, durchsucht den Sendepuffer nach versendbaren Nachrichten und verschickt diese

FC101 - Zuständig für das aufbauen und beenden der Netzwerkverbindung

FC102 - Sorgt dafür das nach einem Spannungsausfall die Netzwerkverbindung sauber zurückgesetzt wird

FC103 - Schreibt die Nachrichten in den Sendepuffer


### Funktionsbausteine

Sämtliche FB's sind Kommunikationsbausteine aus der Siemens Standard Library.

FB65 - TCON stellt eine Verbindung mit einem UDP-Endpunkt her

FB66 - TDISCON beendet die Verbindung 

FB67 - TUSEND verschickt die Daten an den UDP-Endpunkt 

### Datenbausteine

DB3 - Beinhaltet die versendbaren Text-Strings

DB100 - DB für die Variablen des log_master's

DB101 - DB für den Sendepuffer

DB102 - Instanz DB für TCON

DB103 - DB aus UDT65 für die Parameter der TCON Verbindung

DB104 - Instanz DB für TUSEND

DB105 - DB aus UDT66 für die IP-Adresse des UDP-Endpunktes

DB106 - Zweiter Instanz DB für TCON 

DB107 - Instanz DB für TDISCON

DB108 - Zweiter Instanz DB für TDISCON

DB109 - Datenbaustein für die Zuweisung der IP und der local device ID

DB444 - DB zur Demonstration einer Variable als Wert für Parameter 1

UDT65 - Vorlage für die TCON DB's 

UDT66 - Bausteinvorlage für die Adressierung


[1]: http://support.automation.siemens.com/WW/llisapi.dll?func=cslib.csinfo&lang=de&objid=51339682&caller=view
[2]: http://support.automation.siemens.com/WW/llisapi.dll?func=cslib.csinfo&objId=23673044&load=treecontent&lang=en&siteid=cseus&aktprim=0&objaction=csview&extranet=standard&viewreg=WW

[hw]:     hw_config.png
[ipdb]:   ip_db.png
[dbinit]: db_init.png
[ipvat]:  ip_vat.png
[db3]:    db_3.png
[dblist]: db_list.png
[mit]:    http://opensource.org/licenses/MIT
[plclog]: http://plclog.io

