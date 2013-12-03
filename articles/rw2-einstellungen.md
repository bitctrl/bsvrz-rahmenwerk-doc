# Einstellungen

## Hintergrund

Das Rahmenwerk bietet die Möglichkeit, Einstellungen, die allgemein oder nutzerspezifisch gelten, zu speichern und wieder abzurufen.

Einstellungen können entweder lokal hinterlegt oder netzwerkweit abgespeichert werden. 
Zum netzwerkweiten Speichern von Einstellungen werden die Daten in den bisher verwendeten Datenverteiler-Attributgruppen 
"atg.benutzerEinstellungenOberflächeNetzweit" bzw. "atg.globaleEinstellungenOberflächeNetzweit" abgelegt.
Die Attributgruppen sind einer Instanz vom Typ "Oberfläche" zugeordnet, der auch durch den Typ "Autarke Organisationseinheit" 
erweitert wird.

Potentiell vorgesehen ist die Speicherung von Informationen für folgende Gruppierungen:

- allgemeine systemweite Einstellungen
- allgemeine lokale Einstellungen
- benutzerklassenspezifische systemweite Einstellungen
- benutzerklassenspezifische lokale Einstellungen
- benutzerspezifische systemweite Einstellungen
- benutzerspezifische lokale Einstellungen

Eine Reihenfolge der Überlagerung bzw. der Kombinationen der oben genannten Gruppierungen für eine bestimmte Einstellung 
ist nicht vorgesehen. Diese Entscheidung wird durch die jeweilige Anwendung getroffen und kann damit für jede Einstellung 
individuell anders sein.

Die aktuelle Implementierung unterstützt bisher noch nicht die Speicherung der Einstellungen für Benutzerklassen, 
dazu sollte zunächst die Zuordnung der Einstellungen für Benutzerklassen bzw. Benutzer an die Objekte dieses Typs 
verschoben werden. Ein entsprechender Änderungsantrag an die NERZ wurde vorgenommen.  

Der Zugriff auf Einstellungen sollte nicht direkt über die Daten aus den Attributgruppen erfolgen, stattdessen steht 
ein Service zur Verfügung, mit dem die erforderlichen Informationen über das Rahmenwerk gelesen bzw. geschrieben werden können.

## Der Service "Einstellungen"

### Allgemeine Funktionalität

Der Service wird beim Start des Rahmenwerks initialisiert und steht damit unmittelbar den Plug-ins, die ihn verwenden 
wollen, zur Verfügung. Der Zugriff auf die Services des Rahmenwerks ist im Kapitel "Rahmenwerk-Services" detailliert beschrieben.

Der Service selbst wird durch folgende Schnittstelle definiert:

```java
public interface Einstellungen {
    
	Map<String, String> getEinstellungsId(SpeicherKey key) throws IOException;

	Object getValue(final EinstellungsAdresse adresse) throws IOException;

	void setValue(final EinstellungsAdresse adresse, final Object einstellung) throws IOException;

	void setValue(final EinstellungsAdresse adresse, final Object einstellung, final UrlasserInfo urlasser)
				throws IOException;

	void removeValue(EinstellungsAdresse adresse);

	void addEinstellungsListener(final EinstellungChangeListener listener);

	void addEinstellungsListener(final EinstellungChangeListener listener, final String category);

	void removeEinstellungsListener(final EinstellungChangeListener listener);

	void removeEinstellungsListener(final EinstellungChangeListener listener, final String category);

	void addEinstellungsAvailabilityListener( final EinstellungAvailabilityListener listener);

	void removeEinstellungsAvailabilityListener( final EinstellungAvailabilityListener listener);
}
```

#### getEinstellungsId

Die Funktion liefert eine Menge der ID für die Einstellungen und die zugehörigen Typen, 
die für den angegebenen SpeicherKey im Einstellungsspeicher verfügbar sind.

Der Schlüssel in der Map entspricht der Id der jeweiligen Einstellung,
der Wert entspricht dem Typ. Der Typ kann *null* sein, dann
wird für diesen der Standard-Typ *java.lang.String* verwendet.


#### getValue - Lesen einer definierten Einstellung

Die Funktion liefert die mit der angegebenen Einstellungsadresse definierte Einstellung. 

Wenn keine entsprechende Einstellung vorliegt, wird der Wert *null* geliefert.
Es erfolgt keine "Vererbung" der Einstellungen, d.h. wenn nach einer benutzerdefinierten 
Einstellung gefragt wird, wird nicht automatisch die allgemeine Einstellung für die
angegebene ID geliefert, wenn keine benutzerspezifische existiert.  

#### setValue - Setzen einer definierten Einstellung

Die Funktion schreibt den übergebenen Einstellungswert in den Einstellungsspeicher.

Die Standardimplementierung des Rahmenwerks sieht lediglich Strings als Einstellungsobjekte vor.
Sollen Objekte eines anderen Typs als Einstellungen gespeichert werden muss eine entsprechende Factory als Service im Rahmenwerk registriert werden.

Werden Einstellungen netzwerkweit, allgemein gespeichert, erfolgt vor der Übernahme der Parameter
eine Urlasserdatenabfrage zur Authentifikation des Nutzers, dessen Daten werden auch in die 
Urlasserinformationen des Parameters übernommen.

Wird der Urlasserdialog abgebrochen wirft die Funktion *setValue* eine *BenutzerDialogAbgebrochenException*.

Die Abfrage der Urlasserinformationen erfolgt nicht, wenn die set-Funktion mit übergebenen Urlasserdaten
verwendet wird.

#### removeValue - Entfernen einer Einstellung

Die Funktion entfernt die mit der übergebenen Adresse spezifizierte Einstellung unwiederbringlich vom Einstellungsspeicher. 

#### add/removeEinstellungsListener - Listener für Änderungen von Einstellungen

Die Funktionen ermöglichen die Anmeldung von Listenern, die benachrichtigt werden, wenn sich Einstellungen ändern.
Optional kann die Anmeldung auf spezielle Typen erfolgen, das ist aber erst dann sinnvoll, wenn Plug-ins auch 
Factories zum Speichern von Einstellungen, die keine Strings sind, implementieren.

Ein Listener für Einstellungen implementiert folgende Schnitstelle: 

```
public interface EinstellungChangeListener {

    /**
     * es wurde eine neue Einstellung angelegt.
     * 
     * @param event
     *            die Daten des Ereignisses
     */
    void einstellungAngelegt(final EinstellungsEvent event);
    
    /**
     * eine bestehende Einstellung wurde aktualisiert.
     * 
     * @param event
     *            die Daten des Ereignisses
     */
    void einstellungAktualisiert(final EinstellungsEvent event);

    /**
     * eine Einstellung wurde aus dem Einstellungsspeicher entfernt.
     * 
     * @param event
     *            die Daten des Ereignisses
     */
    void einstellungEntfernt(final EinstellungsEvent event);
}
```

und wird damit benachrichtigt über:

- neue Einstellungen
- geänderte Einstellungen und
- entfernte Einstellungen

#### add/removeEinstellungsAvailabilityListener - Listener für die Verfügbarkeit des Einstellungs-Services

Die Funktion erlaubt die Registrierung von Listenern die über die Verfügbarkeit der
netzwerkweiten Einstellungsspeicher informieren.

```java
public interface EinstellungAvailabilityListener {
    
    /** der Einstellungsspeicher ist verfügbar. */
    void available();
    
    /** der Einstellungsspeicher ist nicht verfügbar. */
    void disabled();
}
```

Alle oben aufgeführten Einstellungsspeicher sind nur verfügbar, wenn eine Datenverteilerverbindung besteht.
Im Offline-Betrieb ist nur der allgemeine lokale Einstellungsspeicher verfügbar, da eine Zuordnung zu Benutzerklassen
bzw. Benutzern auf Grund der fehlenden Authentifizierung durch die Datenverteilerkonfiguration nicht möglich ist. 
  
### Die Einstellungsadresse

Die Einstellungsadresse spezifiziert, auf welche Einstellung zugegriffen werden soll.

Momentan wird eine Einstellungsadresse über den Konstruktor:

```java
public EinstellungsAdresse(final String typ, final String id,
    final EinstellungOwnerType ownerType, final String pid,
    final EinstellungLocation location)
```
            
erzeugt. Zusätzliche Convenience-Funktionen sind geplant.

Die Parameter des Konstruktors haben folgende Bedeutung:

#### typ 
Der Objekttyp, mit dem Einstellungen abgespeichert werden, ist prinzipiell freigestellt und wird mit dem 
Parameter "typ" übergeben. Standardmäßig werden alle Parameter als "String" abgelegt. 
Wenn für den Parameter "typ" der Wert *null* übergeben wird, wird der Parametertyp auf "java.lang.String" gesetzt.

Für jeden Typ muss eine entsprechende Factory als OSGI-Service registriert werden, die die Serialisierung
und Deserialisierung des Einstellungsobjekts implementiert. Mit dem Rahmenwerk mitgeliefert wird die Factory
für den Typ "java.lang.String"!

#### id
Die ID unter der die Einstellung innerhalb des Einstellungsspeichers hinterlegt ist. Die Wahl der ID ist nicht weiter festgelegt. 
Es sollte jedoch beachtet werden, dass die Gültigkeit für das gesamte System besteht und dass eine möglichst eindeutige 
ID gewählt werden sollte bspw. unter Einbeziehung der ID des Plug-ins von dem eine Einstellung verwendet wird.

#### ownerType
Der Parameter beschreibt, für wen die Einstellung gültig ist. Mit dem enum stehen folgende Werte zur Verfügung:

```
public enum EinstellungOwnerType {
    /** Systemweite (allgemeine) Einstellung. */
    SYSTEM,
    /** Einstellung ist einer Benutzerklasse zugeordnet. */
    BENUTZERKLASSE,
    /** Einstellung ist einem Benutzer zugeordnet. */
    BENUTZER;
}
```

Zu beachten ist, dass die Einstellungen für Benutzerklassen (noch) nicht unterstützt werden.

#### pid
Der Parameter ist die PID des Benutzers oder der Benutzerklasse für den die Einstellung gültig ist. Für allgemeine Einstellungen
ist der Wert *null* zu übergeben.

#### location
Dieser Parameter ist der Ort, an dem die Einstellung gespeichert werden soll. Mit dem enum stehen folgende Werte zur Verfügung:

```
public enum EinstellungLocation {
    /** Einstellung wird netzwerkweit (als Parameter in Datenverteiler) gespeichert. */
    NETZWERKWEIT, 
    /** Einstellung wird lokal gespeichert. */
    LOKAL;
}
``` 

## Erweiterte Funktionalität - EinstellungsFactory

Als Einstellungen werden in der Standardimplementierung lediglich String-Objekte unterstützt. 

Wenn Objekte anderer Typen direkt im Einstellungsspeicher abgelegt werden sollen, muss in der Einstellungsadresse
der zugeordnete Typ angegeben werden (das kann der Klassenname oder jede andere beliebige ID sein) und es muss eine 
EinstellungsFactory als Service registriert werden, mit der das entsprechende Einstellungsobjekt serialisiert und
deserialisiert werden kann.

Das Interface für die Factory wird wie folgt beschrieben:

```java
public interface EinstellungsFactory {

    /**
     * liefert den Typ der Einstellung (i.d.R. der Klassenname der zu
     * erzeugenden Instanzen)
     * 
     * @return den Name
     */
    String getTyp();

    /**
     * wandelt ein Objekt in eine String-Repräsentation um.
     * 
     * @param einstellung
     *            das zu serialisierende Einstellungsobjekt
     * @return die Stringrepräsentation
     * @throws IOException
     *             die Serialisierung konnte nicht erfolgreich ausgeführt werden
     */
    String serialisiere(final Object einstellung) throws IOException;

    /**
     * wandelt einen String in das gewünschte Einstellungsobjekt um.
     * 
     * @param daten
     *            die Daten des Zielobjekts als String
     * @return das erzeugte Objekt
     * @throws IOException
     *             das Objekt konnte aus dem übergebenen String nicht erzeugt
     *             werden
     */
    Object deserialisiere(final String daten) throws IOException;
}
```
    
Die mit der Funktion *getTyp* gelieferte ID muss dann beim Lesen und Schreiben der Einstellung dem in
der *Einstellungsadresse* angegebenen Typ entsprechen. 
