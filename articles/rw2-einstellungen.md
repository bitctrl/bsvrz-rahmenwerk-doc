Title:   Rahmenwerk 2.0 - Einstellungen  
Author:  U. Peuker  
Date:    August 30, 2013  
Comment: Verwendung des Einstellungsspeichers im Rahmenwerk 2.0

# Einstellungen

## Hintergrund

Das Rahmenwerk bietet die M�glichkeit, Einstellungen die allgemein oder nutzerspezifisch gelten zu speichern und wieder abzurufen.

Einstellungen k�nnen entweder lokal hinterlegt oder netzwerkweit abgespeichert werden. 
Zum netzwerkweiten Speichern von Einstellungen werden die Daten in den bisher verwendeten Datenverteiler-Attributgruppen "atg.benutzerEinstellungenOberfl�cheNetzweit" bzw. "atg.globaleEinstellungenOberfl�cheNetzweit" abgelegt. Die Attributgruppen sind einer Instanz vom Typ "Oberfl�che" zugeordnet, der auch die durch den Typ "Autarke Organisationseinheit" erweitert wird.

Potentiell vorgesehen ist die Speicherung von Informationen f�r folgende Gruppierungen:

- allgemeine systemweite Einstellungen
- allgemeine lokale Einstellungen
- benutzerklassenspezifische systemweite Einstellungen
- benutzerklassenspezifische lokale Einstellungen
- benutzerspezifische systemweite Einstellungen
- benutzerspezifische lokale Einstellungen

Eine Reihenfolge der �berlagerung bzw. der Kombinationen der oben genannten Gruppierungen f�r eine bestimmte Einstellung ist nicht vorgesehen. Diese Entscheidung wird durch die jeweilige Anwendung getroffen und kann damit f�r jede Einstellung individuell anders sein.

Die aktuelle Implementierung unterst�tzt bisher noch nicht die Speicherung der Einstellungen f�r Benutzerklassen, dazu sollte zun�chst die Zuordnung der Einstellungen f�r Benutzerklassen bzw. Benutzer an die Objekte diesen Typs verschoben werden. Ein entsprechender �nderungsantrag an die NERZ wurde vorgenommen.  

Der Zugriff auf Einstellungen sollte nicht direkt �ber die Daten aus den Attributgruppen erfolgen, stattdessen steht ein Service zur Verf�gung, mit dem die erforderlichen Informationen �ber das Rahmenwerk gelesen bzw. geschrieben werden k�nnen.

## Der Service "Einstellungen"

### Allgemeine Funktionalit�t

Der Service wird beim Start des Rahmenwerk initialisiert und steht damit unmittelbar den Plug-ins, die ihn verwenden wollen zur Verf�gung. Der Zugriff auf die Services das Rahmenwerks ist im Kapitel "Rahmenwerk-Services" detailliert beschrieben.

Der Service selbst wird durch folgende Schnittstelle definiert:

```java
    public interface Einstellungen {
    
        Object getValue(final EinstellungsAdresse adresse) throws IOException;
        
        void setValue(final EinstellungsAdresse adresse, final Object einstellung) throws IOException;
        void setValue(final EinstellungsAdresse adresse, final Object einstellung, final UrlasserInfo urlasser) throws IOException;

        void removeValue(EinstellungsAdresse adresse);
        
        void addEinstellungsListener(final EinstellungChangeListener listener);
        void addEinstellungsListener(final EinstellungChangeListener listener, final String category);
        void removeEinstellungsListener(final EinstellungChangeListener listener);
        void removeEinstellungsListener(final EinstellungChangeListener listener, final String category);

        void addEinstellungsAvailabilityListener(final EinstellungAvailabilityListener listener);
        void removeEinstellungsAvailabilityListener(final EinstellungAvailabilityListener listener);
    }
```
 
#### getValue - Lesen einer definierten Einstellung

Die Funktion liefert die mit der angegebenen Einstellungsadresse definierte Einstellung. 

Wenn keine entsprechende Einstellung vorliegt, wird der Wert *null* geliefert.
Es erfolgt keine "Vererbung" der Einstellungen, d.h. wenn nach einer benutzerdefinierten Einstellung gefragt wird, wird nicht automatisch die allgemeine EInstellung f�r die angegebene ID geliefert, wenn keine benutzerspezifische existiert.  

#### setValue - Setzen einer definierten Einstellung

Die Funktion schreibt den �bergebenen Einstellungswert in den Einstellungsspeicher.

Die Standardimplementierung des Rahmenwerks sieht lediglich Strings als Einstellungsobjekte vor.
Sollen Objekte eines anderen Typs als Einstellungen gespeichert werden muss eine entsprechende Factory als Service im Rahmenwerk regsitriert werden.

#### removeValue - Entfernen einer Einstellung

Die Funktion entfernt die mit der �bergebenen Adresse spezifizierte Einstellung unwiederbringlich vom Einstellungsspeicher. 

#### add/removeEinstellungsListener - Listener f�r �nderungen von Einstellungen

Die Funktionen erm�glichen die Anmeldung von Listenern, die benachrichtigt werden, wenn sich Einstellungen �ndern.
Optional kann die Anmeldung auf spezielle Typen erfolgen, das ist aber erst dann sinnvoll, wenn Plug-ins auch Factories zum Speichern von Einstellungen, die keine Strings sind implementieren.

Ein Listener f�r Einstellungen implementiert folgende Schnitstelle 

```java
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

und wird damit benachrichtigt �ber:

- neue Einstellungen
- ge�nderte Einstellungen und
- entfernte Einstellungen

#### add/removeEinstellungsAvailabilityListener - Listener f�r die Verf�gbarkeit des Einstellungs-Services

Die Funktion erlaubt die Registrierung von Listenern die �ber die Verf�gbarkeit der
netzwerkweiten Einstellungsspeicher informieren.

```java
    public interface EinstellungAvailabilityListener {
    
        /** der Einstellungsspeicher ist verf�gbar. */
        void available();
    
        /** der Einstellungsspeicher ist nicht verf�gbar. */
        void disabled();
    }
```

Alle oben aufgef�hrten Einstellungsspeicher sind nur verf�gbar, wenn eine Datenverteilerverbindung besteht.
Im Offline-Betrieb ist nur der allgemeine lokale Einstellungsspeicher verf�gbar, da eine Zuordnung zu Benutzerklassen bzw. Benutzern auf Grund der fehlenden Authentifizierung durch die Datenverteilerkonfiguration nicht m�glich ist. 
  
### Die Einstellungsadresse

Die Einstellungsadresse spezifiziert, auf welche Einstellung zugegriffen werden soll.

Momentan wird eine Einstellungsadresse �ber den Konstruktor:

```java
    public EinstellungsAdresse(final String typ, final String id,
            final EinstellungOwnerType ownerType, final String pid,
            final EinstellungLocation location)
```
            
erzeugt. Zus�tzliche Convenience-Funktionen sind geplant.

Die Parameter des Konstruktors haben folgende Bedeutung:

#### typ 
der Objekttyp mit dem Einstellungen abgespeichert werden ist prinzipiell freigestellt und wird mit dem Parameter "typ" �bergeben. Standardm��ig werden alle Parameter als "String" abgelegt. Wenn f�r den Parameter "typ" der Wert "*null*" �bergeben wird, wird der Parametertyp auf "java.lang.String" gesetzt.

F�r jeden Typ muss eine entsprechende Factory als OSGI-Service registriert werden, die die Serialisierung und Deserialisierung des Einstellungsobjekts implementiert. Mit dem Rahmenwerk mitgeliefert wird die Factory f�r den Typ "java.lang.String"!

#### id
die ID unter der die Einstellung innerhalb des Einstellungsspeichers hinterlegt ist. Die Wahl der ID ist nicht weiter festgelegt. Es sollte jedoch beachtet wedren, das die G�ltigkeit f�r das gesamte System besteht und das eine m�glichst eindeutige ID gew�hlt werden sollte bspw. unter Einbeziehung der ID des Plug-ins von dem eine Einstellung verwendet wird.

#### ownerType
beschreibt, f�r wen die Einstellung g�ltig ist. Mit dem enum stehen folgende Werte zur Verf�gung:

```java
    public enum EinstellungOwnerType {

        /** Systemweite (allgemeine) Einstellung. */
        SYSTEM,
        /** Einstellung ist einer Benutzerklasse zugeordnet. */
        BENUTZERKLASSE,
        /** Einstellung ist einem Benutzer zugeordnet. */
        BENUTZER;
    }
```    
Zu beachten ist, das die Einstellungen f�r Benutzerklassen (noch) nicht unterst�tzt werden.

#### pid
ist die PID des Benutzers oder der Benutzerklasse f�r den die Einstellung g�ltig ist. F�r allgemeine Einstellungen ist der Wert *null* zu �bergeben.

#### location
ist der Ort, an dem die Einstellung gespeichert werden soll. Mit dem enum stehen folgende Werte zur Verf�gung:

```java
    public enum EinstellungLocation {
    
        /** Einstellung wird netzwerkweit (als Parameter in Datenverteiler) gespeichert. */
        NETZWERKWEIT, 
    
        /** Einstellung wird lokal gespeichert. */
        LOKAL;
    } 
```

## Erweiterte Funktionalit�t - EinstellungsFactory

Als Einstellungen werden in der Standardimplementierung lediglich String-Objekte unterst�tzt. 

Wenn Objekte anderer Typen direkt im Einstellungsspeicher abgelegt werden sollen, muss in der Einstellungsadresse der zugeordnete Typ angegeben werden (das kann der Klassenname oder jede andere beliebige ID sein) und es muss eine EinstellungsFactory als Servive registriert werden, mit der das entsprechende Einstellungsobjekt serialisiert und deerialisiert werden kann.

Das Interface f�r die Factory wird wie folgt beschrieben:

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
        * wandelt ein Objekt in eine String-Repr�sentation um.
        * 
        * @param einstellung
        *            das zu serialisierende Einstellungsobjekt
        * @return die Stringrepr�sentation
        * @throws IOException
        *             die Serialisierung konnte nicht erfolgreich ausgef�hrt werden
        */
        String serialisiere(final Object einstellung) throws IOException;

        /**
        * wandelt einen String in das gew�nscht Einstellungsobjekt um.
        * 
        * @param daten
        *            die Daten des Zielobjekts als String
        * @return das erzeugte Objekt
        * @throws IOException
        *             das Objekt konnte aus dem �bergebenen String nicht erzeugt
        *             werden
        */
        Object deserialisiere(final String daten) throws IOException;
    }
```
    
Die mit der Funktion *getTyp* gelieferte ID muss dann beim Lesen und Schreiben der Einstellung dem in der *Einstellungsadresse* angegebenen Typ entsprechen. 


