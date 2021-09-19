Ich übernehme keine Haftung für Schäden, die aufgrund der Nutzung 
der Software entstehen oder entstanden sind. 
Jeder ist für sein tun selber verantwortlich.
Mir sind keine Probleme bekannt.

<img src="Soyosource.png" alt="Soyosource Controller by BavarianSuperGuy"/>

# EspSoyosourceController mit Webseite
Der Esp8266 ist über ein Rs485 Modul mit dem Soyosource 1200 Solar/Batterie Wechselrichter verbunden und kann diesem die Leistungswerte senden die er im Stromnetz ausgleichen/einspeisen soll. 
ManualMode: Der Leistungswert kann auf der Esp Webseite eingegeben und gesendet werden. 
AutoMode: Der Esp empfängt über Mqtt Topic oder WebInterface den LeistungsWert (z.B. der Wert einer Messeinrichtung über die 3 Phasen)
der gewünschten zu regelnden Leistung und regelt den Soyosource automatisch.

Kurzum die Firmware die es hier zum Download gibt, ersetzt den Limiter.

Die Firmware(firmware_vX.X.X.X.bin") ist im Verzeichnis espflasher.
Diese kann unkompliziert auf einen 4Mbyte Esp mit dem im Verzeichnis enhaltenen
Tool "FlashESP8266.exe" geflasht werden. 
Dieses Tool FlashESP8266.exe ist nicht von mir , aber hat schon hevorragende Dienste geleistet.

<img src="Webseite SOYOSOURCE Controller.png" alt="Soyosource Controller by BavarianSuperGuy"/>

Was kann der ESP8266 Soyosource Controller :

- annähernd Nulleinspeisung

Grundlegend ist FAKT: wann der SOYO Wechselrichter keine Signale über Rs485 gesendet bekommt hört er auf einzuspeisen(Safety)!

- Der Esp nimmt die Leistung die er Regeln soll über Mqtt und Webschnittstelle auf.
- Zu übergebenes Zahlenformat 200 oder 200.00
- Der ESP nimmt die Leistung an und übergibt Sie dem Soyo , danach ist eine 3Sekunden Pause
- Erst dann wird der zuletzt empfangene Leistungswert dem Soyo übergeben dann wieder 3 sekunden Pause...
- Damit geschieht eine saubere einpendelung gegen ~0watt Verbrauch bzw. er regelt sich in einen vorgegebenen Toleranzbereich ein.
- In der ESP Web Seite(Ipaddresse des Esp im Browser eingeben) kann man per Button die Einspeisung beenden(Notaus), schicke explizit 0watt an SOYO , geht schneller als zu warten bis er selbst beendet da er anschliessend auch keine Signale mehr bekommt.
- In der Web Seite kann man per Button die maximale Einspeiseleistung eingestellt werden(MaxPower)
- Die blinkende Überschrift in der Webseite ist ein Alive zeichen , solange die Überschrift "ESP Herzschlag" blinkt , ist alles gut
- Wann der Soyosource einspeisst , blinkt die DCZeile in der Esp Webseite
- Derzeit pendelt der Esp sich im Toleranz Bereich -20 |0| +5 Watt ein , geplant ist die Toleranzwerte einzustellen zu können wann jemand gar keine -watts(Einspeisung ins Netz) sehen will.
- Doppelreset implementierung um wieder ins Configportal im eigenen Esp AP zu kommen
- ElegantOta Implementierung für FirmwareUpdates 
(Für UPDATE: Wann der ESP in Betriebsart Httpclient eingestellt ist , vor update auf eine ruhige zone einstellen z.B. statisch Mqtt
nach Update dann wieder zurückstellen und Übernehmen)
- Betriebs Modi's:
# Automatik HttpInterface
# Automatik Mqtt
# Automatik HttpClient(Shelly) mit Angabe ShellyIpAddresse greift auf die api "http://xxx.xxx.xxx.xxx/emeter/0 -2" zu
# Statisch HttpInterface und WebGui
# Statisch Mqtt(derzeit noch keine Funktion)

- ...




RS485 ttl Adapter (2 verschiedene getestet)
- rs485 adapter A an A von Soyo anschliessen und B an B

<img src="rs485.png" alt="Soyosource Controller by BavarianSuperGuy"/>
- Beim "DI DE RE RO" Rs485 Modul
werden die mittleren Rs485 Pins "DE RE" auf einen Pin gebrückt an Esp GPIO0 angeschlossen.
Dann Rs485 "DI" auf Esp TX  und Rs485 "RO" auf Esp RX 

<img src="rs485_2 .png" alt="Soyosource Controller by BavarianSuperGuy"/>
- Beim RX TX Rs485 Modul
wird der Rs485 TX mit Esp TX verbunden und Rs485 RX mit Esp RX verbunden , also nicht kreuzen!

!Vcc ist bei beiden Modulen 3.3volt!

Keinen USB/TTL Adapter als 3,3v Poduktiv Stromquelle verwenden, für erstflashen is es OK!

ESP:

EINRICHTUNG##############################################################################################
- 0.Bei Nutzung von Mqtt: Mqtt.fx Client öffnen mit Broker verbinden und Topic : 'Soyosource/#' abonnieren
- 0.1 Es kommt nach Schritt 8. ein Info Publish vom Esp mit der IP adresse.
- 0.2 Man kann die Ip Adresse nat. auch auf dem herkömmlichen Weg herausfinden
- 1.Firmware auf den gelöschten Esp8266 flashen
- 2.Esp Neustart
- 3.Im Wlan nach SOYO_... suchen
- 4.Wlan mit Pwd 12345678 verbinden
- 5.Es öffnet sich automatisch Browser Fenster mit 192.168.4.1
- 6.Unter Configuration
- 6.1 Wifi auswählen oder eingeben und Passwort eingeben
- 6.2 Mqtt kann derzeit nur ohne ssl und ohne Benutzer/Passwort benutzt werden
- 7.Speichern mit dem Button ganz unten
- 8.Esp Neustart
- 9.Ipadresse in Mqtt Client oder Router ausfindig machen
EINRICHTUNG#############################################################################################

INBETRIEBNAHME##########################################################################################
- 1.Ipaddresse im browser aufrufen
- 2.L1L2L3 Topic von geöffneter Webseite in deinen iobroker/nodered ... übernehmen
ODER wann Nutzung von Mqtt aus ist :
- 3.IPADRESS:80/L1L2L3?Value=200.66&Mode=SetAuto
- 3.1 IPADRESS durch die ip vom ESP ersetzen, 200.66 stellt den kummulierten Wert deiner 2/3 phasen da
INBETRIEBNAHME##########################################################################################

Tip:
-  nur leistungsdaten über L1L2L3 (egal welches Interface) senden
wann sich was geändert hat ! Der esp ansich schickt schon jede sekunde den zuletzt angenommenen Wert.

- 1.Mqtt L1L2L3 Publish Setting : Qos2 und Retain! ... kein muss
- 2.Wenn WebInterface LL1L2L3?Value=200.66&Mode=SetAuto... genutzt wird dann schon alle sekunden senden


Falls Ihr mich unterstützen wollt, Spenden sind herzlich willkommen und würde mich sehr freuen :-)
Hier bitte - https://paypal.me/armerprogrammer
