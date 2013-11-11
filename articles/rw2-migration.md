Migrations-Hinweise
===================

Die Migration eines bestehenden Plug-ins erfordert im wesentlichen folgende
Maßnahmen:

- Anpassen der Abhängigkeiten der Rahmenwerks-Plug-ins (Versionierung)
- Ersetzen der Singletons für Einstellungen, Berechtigungen und Datenverteilerverbindung
  durch die entsprechenden Services des Rahmenwerks
- Umstellung von Actions in ActionsSets auf Commands
- Definition von Berechtigungen per ExtensionPoint

## Migrationssupport

Im ersten Schritt kann in einem bestehenden Plug-in die Abhängigkeit zum Plug-in
*de.bsvrz.buv.rw.migrationsupport* ergänzt werden.

Damit sollte der bestehende Code in den meisten Fällen kompilierbar sein.

Folgende bekannte zusätzliche Eingriffe sind zusätzlich potentiell erforderlich: 

- das zum Drucken verwendete Projekt *Paperclips* wurde von SourceForge zu einem Eclipse 
  Nebula Projekt übernommen. Dabei wurde der Name und die Packages von net.sf.* auf 
  org.eclipse.* geändert und die Version von 1.x auf 2.x angehoben.
  Die Imports in betroffenen Paketen müssen aktualisiert werden.
- Das Plugin org.junit4 gibt es nicht mehr. Es gibt nur noch org.junit. Das aber in 
  Version 4.x. Abhängigkeiten zu diesem Plug-in müssen entsprechend angepasst werden.
- die Schnittstelle *ILegende* hat zwei neue Methoden, die eventuell in den betroffenen
  Klassen nachgeführt werden müssen.
  
Der **Migrationssupport** ist kein offiziell unterstütztes Plug-in für den 
Produktiveinsatz sondern dient lediglich in dem Sinne der Erleichterung der Portierung, 
das der Code im Wesentlichen kompilierbar bleibt und mit hoher Wahrscheinlichkeit auch
funktioniert. 

Der Entwickler hat dann die Möglichkeit schrittweise die veralteten Komponenten und 
Funktionen durch die neuen zu ersetzen.

**Eine Portierung ohne Migrationssupport ist in jedem Fall angeraten!**

## Anpassungen ohne Migrationssupport

- die API des Urlasserdialogs hat sich geändert (neuer Klassenname *UrlasserDatenDialog*).
- die API der Rahmenwerkseinstellungen hat sich geändert
- die API für das Drucken hat sich geändert (Rahmenwerk-API für die Nutzung von Paperclips).
- der Extension Point org.eclipse.ui.actionSets ist deprecated und muss durch 
  org.eclipse.ui.commands ersetzt werden.
- der Extension Point org.eclipse.ui.viewActions ist deprecated und muss durch 
  org.eclipse.ui.menus ersetzt werden.

### Urlasserdialog

Der Urlasserdialog wird jetzt wie ein normaler Dialog verwendet und das Ergebnis nach 
dem Schließen an diesem abgefragt. Es gibt keine Methode mehr die den Dialog öffnet 
und das Ergebnis zurück gibt. 

Stattdessen wird dem Urlasserdialog im Konstruktor eine *UrlasserDatenSender* übergeben, 
der die entsprechenden Aufgaben in der Kommunikation mit dem Datenverteiler übernimmt.

> Zum Thema "Urlasserdialog" gibt es ein gesondertes Kapitel! 

### Drucken

Der Name der Schnittstelle *IDruckvorschau* ist jetzt *RwPrintable* und liegt in 
einem anderen Package. Die Schnittstellen beiden Interfaces sind identisch, d.h. es
müssen lediglich die Namen korrigiert und Imports angepasst werden.

### DialogEreignisSystemkalenderUebernahme

Der Dialog wurde in einen normalen JFace-Dialog umgebaut und umbenannt. Die Verwendung muss entsprechend 
angepasst werden:

**ALT**
```
DialogEreignisSystemkalenderUebernahme dialog = new DialogEreignisSystemkalenderUebernahme(
					shell);
LinkedList<IEintragBereich> bereiche 
             = dialog.oeffnen(DialogEreignisSystemkalenderUebernahme.Quelle.systemkalender);
```

**NEU**
```
KalenderBereichDialog bereichsDialog = new KalenderBereichDialog(shell, Quelle.systemkalender);
if ( bereichsDialog.open() == KalenderBereichDialog.OK) {
	LinkedList<IEintragBereich> bereiche = bereichsDialog.getSelektierteBereiche();
}
```

### EinstellungsArt

Das enum *EinstellungsArt* wurde ersetzt, und durch eine Klasse *Speicherkey* ersetzt,
da damit bisher nur die Speicherziele:

- Benutzereinstellungen lokal (angemeldeter Nutzer)
- Benutzereinstellungen netzwerkweit (angemeldeter Nutzer)
- Allgemeine Einstellungen lokal
- Allgemeine Einstellungen netzwerkweit 
 
abgedeckt wurden
 
Potentiell können aber auch Einstellungen für

- Berechtigungsgruppen lokal
- Berechtigungsgruppen netzwerkweit
- Benutzereinstellungen lokal (irgendein Nutzer)
- Benutzereinstellungen netzwerkweit (irgendein Nutzer)

gelesen und gespeichert werden.

Die Klasse *Speicherkey* bietet Convenience-Methoden als Ersatz für die enum-Werte.

```
public static SpeicherKey benutzerNetzweit();
public static SpeicherKey benutzerLokal();
public static SpeicherKey allgemeinNetzweit();
public static SpeicherKey allgemeinLokal();

public static Collection<SpeicherKey> getDefaultKeys();
```

Die Methode *getDefaultKeys* bildet den Ersatz für *EInstellungsArt.values()*.

> Zu beachten ist, dass nach dem Verständnis des Rahmenwerks 2.0 nur im Online-Betrieb alle 
> genannten Einstellungsspeicher zur Verfügung stehen, da ansonsten kein authentifizierter
> Benutzer zur Verfügung steht.
> Im Offlinebetrieb kann nur auf **allgemeinLokal** zugegriffen werden, der Speicher steht
> genau genommen hier auch für **offlineEinstellung**.




 