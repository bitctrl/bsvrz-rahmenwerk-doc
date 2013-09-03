Title:   Rahmenwerk 2.0 - Berechtigungen  
Author:  U. Peuker  
Date:    August 14, 2013  
Comment: Darstellung der Definition von Oberflächen-Berechtigungen im Rahmenwerk 2.0

# Berechtigungen

## Hintergrund

Berechtigungen werden im Rahmenwerk als abstrakte Definitionen betrachtet, 
denen bezogen auf eine gegebene Benutzerklasse das Attribut Freigabe oder Sperrung 
zugeordnet werden kann.

Die Funktionen mit Berechtigungen sind insofern abstrakt, dass sie nicht unbedingt 
Funktionen in Sinne von konkreten Aktionen, sondern alternativ auch andere 
Eigenschaften wie Sichtbarkeit, Bedienbarkeit, Filterung bestimmter Eigenschaften 
usw. definieren können.

Eine Oberflächenfunktion definiert sich im Rahmenwerk 2.0 durch folgende Eigenschaften:
- ID der Funktion
- Name der Funktion
- Beschreibung der Funktion
- Kategorie

Oberflächenfunktionen können dynamisch durch die zur Verfügung stehenden API-Funktionen
in die Berechtigungsverwaltung des Rahmenwerks integriert werden. 
Auf welche Art und Weise die Funktionen konkret definiert werden ist durch das Rahmenwerk 
nicht festgelegt. Es kann sich gleichermaßen um Funktionsdefinitionen im Sourcecode eines 
entsprechenden Plug-Ins handeln, wie um Definitionen über Extension-Points 
oder externe Konfigurationsdateien.

**Zu bevorzugen ist jedoch die Definition von ExtensionPoints!**

Die Kategorie der Oberflächenfunktionen dient lediglich der Verbesserung der Möglichkeiten
zur Repräsentation der verfügbaren Definitionen im konkreten Datenverteilersystem.

Für jede Oberflächenfunktion können Berechtigungen pro definierter Benutzerklasse
vergeben werden. Damit werden die in den Technischen Anforderungen für das Segment BuV
geforderten Eigenschaften (TBuV-103/104) wie folgt realisiert:

- **Wer?** - entspricht der festgelegten Benutzerklasse (eine Rechtevergabe auf Benutzerebene ist nicht vorgesehen und wird nur implizit durch die Zuordnung eines Nutzers zu einer Benutzerklasse realisiert)
- **Was?** - entspricht der Oberflächenfunktion an sich, definiert über die festgelegte ID
- **Wie?** - entspricht der Festlegung Freigabe/Sperrung
- **Worauf?** - wird nicht direkt abgebildet sondern bei Bedarf durch mehrere Oberflächenfunktionen umgesetzt, d.h. statt der Definition einer einzelnen Funktion mit Beschränkungen auf jeweils ein Bedienelement eines Plug-Ins definiert das Plug-In zwei Funktionen, die individuell mit Rechte versehen werden können.
- **Ausnahmen?** - auf Ausnahmen im eigentlichen Sinne wird verzichtet, die Zuordnungen Benutzerklasse-Funktion sind eindeutig, die Parametrierung erfolgt über geeignete Editoren, so dass auf Platzhalter in den Definitionen der Rechte verzichtet werden kann und Ausnahmen damit obsolet werden

## Repräsentation im Datenverteiler

Die Oberflächenberechtigungen werden im vorhandenen Parameterdatensatz eines Objekts 
"Oberfäche" abgelegt. In der Regel ist das die AOE, dem Rahmenwerk kann aber ein anderes
Objekt zur Berechtigungsverwaltung zugewiesen werden.

Die entsprechenden Objekte und Attributgruppen sind in der aktuellen 
Datenverteilerkonfiguration verfügbar. Die Einträge des Parameterdatensatzes bezüglich 
einschränkender Objekte, Ausnahmen und Verschachtelungstiefe von Ausnahmen werden ignoriert.

Der Zugriff auf die Berechtigungen erfolgt über einen vom Rahmenwerk bereitgestelltem Service
vom Typ "Berechtigungen".

## Programmierschnittstelle

### Service "Berechtigungen"

Die Schnittstelle Berechtigungen liegt in der folgenen Form vor:

```java
public interface Berechtigungen {

    void addOberflaechenFunktion(final FunktionMitBerechtigung funktion);
    void removeOberflaechenFunktion(final String id);

    Collection<FunktionMitBerechtigung> getFunktionen();
    FunktionMitBerechtigung getFunktion(String id);
    
    boolean hasBerechtigung(SystemObject benutzer,
            FunktionMitBerechtigung funktion);
    boolean hasBerechtigung(String benutzerPid, FunktionMitBerechtigung funktion);
    
    Collection<SystemObject> getBerechtigungsKlassen();

    void addOberflaechenFunktionsListener(IBerechtigungListener listener);
    void addListenerForFunktion(IBerechtigungListener listener,
            FunktionMitBerechtigung funktion);
    void removeListener(IBerechtigungListener listener);
```

Folgende Funktionsgruppen werden zur Verfügung gestellt.

#### add/removeOberflaechenFunktion
Die Funktionen dienen dazu Berechtigungsfunktionen zur internen Berechtigungsverwaltung
des Rahmenwerks hinzuzufügen bzw. zu entfernen.

Diese Funktionalität besteht im Wesentlichen aus Kompatibilitätsgründen. Zu bevorzugen ist
die Definition von ExtensionPoints durch die Plug-ins, die Berechtigungsfunktionen in die
Rahmenwerk-Applikation einbringen wollen. 

#### getFunktion/en
liefert alle bzw. durch die ID festgelegte Funktionen aus der Berechtigungsverwaltung des
Rahmenwerks.

#### hasBerechtigung
prüft, ob die Berechtigung für eine engegebene Berechtigungsfunktion hat.

Für die Prüfung der Berechtigungen wird geprüft, ob eine der Berechtigungsklassen, der der
Nutzer zugeordnet ist eine Freigabe für die übergebene Funktion besitzt.

**Die Zuordnung der Nutzer zu Berechtigungsklassen hängt davon ab, ob das neue oder
alte Berechtigungskonzept verwendet wird.**
  
#### Listener-Funktionen
melden Listener an/ab, die über Änderungen in der Berchtigungsverwaltung allgemein oder
für bestimmte Berechtigungsfunktionen benachrichtigt werden sollen.

### FunktionMitBerechtigung
Instanzen dieser Klasse repräsentieren die in der Berechtigungsverwaltung des Rahmenwerks
registrierten Funktionen, denen eine Berechtigung/Freigabe pro Berechtigunsklasse zugeordnet
wird.

Die Instanzen können zwar aus Kompatibilitätsgründen dirket angelegt werden, sollten aber
normalerweise als ExtensionPoints in Plug-ins definiert und vom Rahmenwerk intern 
initialisiert werden.

```java
public FunktionMitBerechtigung(final String pluginId,
          final String kategorie, final String id, final String bezeichnung,
          final String beschreibung) {
```

Eine Berechtigungsfunktion ist durch die im Konstruktor übergebenen Attribute definiert:

- **pluginId** die ID des Plug-ins in der die Funktion angelegt wird
- **kategorie** eine optionale Kategorie, die nur zu Anzeigezwecken verwendet wird
- **id** die eindeutige ID der Funktion
- **bezeichnung** der Name der Funktion
- **beschreibung** ein optionaler Beschreibungstext für die Funktion

### IBerechtigungsListener
Die Schnittstelle für einen Listener, der über Änderungen in Berechtigungsverwaltung
informiert wird.

```java
public interface IBerechtigungListener {
    void sperrung(BerechtigungEreignis e);
    void freigabe(BerechtigungEreignis e);
}
```

Das BerechtigungsEreignis enthält die Informationen, welche Funktionen gesperrt bzw. 
freigegeben wurden.

```java
public class BerechtigungEreignis extends EventObject {

    ....

    /**
     * ermittelt, ob eine Freigabe gemeldet wird.
     * 
     * @return der Zustand
     */
    public boolean isFreigabe() {
        return freigabe;
    }

    /**
     * liefert die betroffene Berechtigungsklasse.
     * 
     * @return die Berechtigungsklasse
     */
    public SystemObject getBerechtigungsKlasse() {
        return berechtigungsKlasse;
    }

    /**
     * liefert die betroffenen Funktionen.
     * 
     * @return eine Liste der Funktionen
     */
    public List<FunktionMitBerechtigung> getFunktionen() {
        return Collections.unmodifiableList(funktionen);
    }
}
```

## Definition einer Berechtigungsfunktion mittels ExtensionPoint

Die Berechtigungsfunktionen werden innerhalb des Plug-ins, das eine solche Funktion
bereitstellen möchte durch einen ExtensionPoint vom Typ "de.bsvrz.buv.rw.rw.funktionmitberechtigung"
repräsentiert.

Eine Extension "function" wird durch die Attribute:

- **id** die eindeutige ID der Funktion
- **bezeichung** die Bezeichnung der Funktion
- **kategorie** die optionale Kategorie
- **beschreibung** die optionale Beschreibung

definiert.

Eine auf diese Weise angelegte Berechtigungsfunktion wird vom Rahmenwerk automatisch initialisiert und
der Berechtigungsverwaltung zugeordnet.




