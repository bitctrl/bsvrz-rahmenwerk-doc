Title:   Rahmenwerk 2.0 - Migration  
Author:  U. Peuker  
Date:    August 30, 2013  
Comment: Hinweise zur Migration bestehender Rahmenwerks-Plug-ins

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

Im ersten Schritt kann in einem betehenden Plug-in die Abhängigkeit zum Plug-in
*de.bsvrz.buv.rw.migrationsupport* ergänzt werden.

Damit sollte der bestehende Code in den meisten Fällen kompilerbar sein.

Folgende bekannte zusätzliche Eingriffe sind zusätzlich potentiell erfoderlich: 

- das zum Drucken verwendete Projekt *Paperclips* wurde von SourceForge zu einem Eclipse 
  Nebula Projekt übernommen. Dabei wurde der Name und die Packages von net.sf.* auf 
  org.eclipse.* geändert und die Version von 1.x auf 2.x angehoben.
  Die Imports in betroffenen Paketen müssen aktuelisiert werden.
- Das Plugin org.junit4 gibt es nicht mehr. Es gibt nur noch org.junit. Das aber in 
  Version 4.x. Abhängigkeiten zu diesem Plug-in müssen entsprechend angepasst werden.
- die Schnittstelle *ILegende* hat zwei neue Methoden, die eventuell in den betroffenen
  Klassen nachgeführt werden müssen.
  
Der **Migrationssupport** ist kein offziell unterstütztes Plug-in für den 
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
einem anderem Package. Die Schnittstellen beiden Interfaces sind identisch, d.h. es
müssen lediglich die Namen korrigiert und Imports angpasst werden.

 