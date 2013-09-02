Title:   Rahmenwerk 2.0 Services  
Author:  U. Peuker  
Date:    August 11, 2013  
Comment: Darstellung der Basis-Services des Rahmenwerk 2.0,
		 Funktionen und Zugriff  

		 
Rahmenwerk 2.0 Services	
=======================
	 
## Allgemeines

Die Funktionalit�t des Rahmenwerks wird im Wesentlichen �ber OSGI-Services bereitgestellt, 
diese Ersetzen die bisher verwendeten Singletons wie DaVVerbindung, 
EinstellungsSpeicher, Oberfl�chenFunktionen, ...

Der Zugriff auf die Services kann auf verschiedenen Wegen erfolgen:

### Service-Componente

Ein Plug-in, das Rahmenwerk-Services ben�tigt installiert eine Komponente, die von den entsprechenden 
Services abh�ngig ist. Die Komponente wird im Rahmen der Initialisierung des OSGI-Frameworks 
mit den notwendigen Informationen versorgt und aktiviert, wenn alle Voraussetzungen 
f�r ihren Betrieb vorliegen.
 
Beispielhaft ist hier die entsprechende Komponente im Plug-in "Migrationssupport" dargestellt, �ber 
die alle grundlegenden Services des Rahmenwerks bereitgestellt werden (Datei rahmenwerkservice.xml).

```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <scr:component xmlns:scr="http://www.osgi.org/xmlns/scr/v1.1.0" name="de.bsvrz.buv.rw.compatibility">
        <implementation class="de.bsvrz.buv.rw.compatibility.internal.RahmenwerkService"/>
        <reference bind="bindRahmenwerk" cardinality="1..1" interface="de.bsvrz.buv.rw.basislib.Rahmenwerk" name="Rahmenwerk" policy="static" unbind="unbindRahmenwerk"/>
        <reference bind="bindBerechtigungen" cardinality="1..1" interface="de.bsvrz.buv.rw.basislib.berechtigung.Berechtigungen" name="Berechtigungen" policy="static" unbind="unbindBerechtigungen"/>
        <reference bind="bindEinstellungen" cardinality="1..1" interface="de.bsvrz.buv.rw.basislib.einstellungen.Einstellungen" name="Einstellungen" policy="static" unbind="unbindEinstellungen"/>
    </scr:component>
```

Der entsprechende Quellcode sieht dann folgenderma�en aus:

```java
    public class RahmenwerkService {

        private static RahmenwerkService service;
        private Rahmenwerk rahmenWerk;
        private Berechtigungen berechtigungen;
        private Einstellungen einstellungen;

        public static RahmenwerkService getService() {
            return RahmenwerkService.service;
        }

        protected void activate() {
            RahmenwerkService.service = this;
        }

        protected void deactivate() {
            RahmenwerkService.service = null;
        }

        protected void bindRahmenwerk(final Rahmenwerk newRahmenWerk) {
            this.rahmenWerk = newRahmenWerk;
        }

        @SuppressWarnings("unused")
        protected void unbindRahmenwerk(final Rahmenwerk oldRahmenWerk) {
            this.rahmenWerk = null;
        }

        public Rahmenwerk getRahmenWerk() {
            return rahmenWerk;
        }

        protected void bindBerechtigungen(final Berechtigungen newBerechtigungen) {
            this.berechtigungen = newBerechtigungen;
        }

        @SuppressWarnings("unused")
        protected void unbindBerechtigungen(final Berechtigungen oldBerechtigungen) {
            this.berechtigungen = null;
        }

        public Berechtigungen getBerechtigungen() {
            return berechtigungen;
        }

        protected void bindEinstellungen(final Einstellungen newEinstellungen) {
            this.einstellungen = newEinstellungen;
        }

        @SuppressWarnings("unused")
            protected void unbindEinstellungen(final Einstellungen oldEinstellungen) {
            this.einstellungen = null;
        }

        public Einstellungen getEinstellungen() {
            return einstellungen;
        }
    }
```

Die Komponente muss noch in in der MANIFEST.MF eingebunden werden:

    ....
    Service-Component: OSGI-INF/rahmenwerkservice.xml
    ....     

F�r die Definition und Einbindung der Komponenten setehen in der Eclipse-IDE entsprechende Hilfsmittel zur Verf�gung!
 
Das Plug-in, das die Komponente installiert, hat damit volle Kontrolle �ber die Verf�gbarkeit 
der notwendigen Services.   
 
### Zugriff �ber den Bundle-Aktivator

Der Aktivator eines Bundles (Plug-ins) hat Zugriff auf die Service-Registry des Rahmenwerks.
Aus dieser kann der gew�nschte Service ermittelt werden. 

Nachstehend wird beispielhaft der verf�gbare Service "*Rahmenwerk* ermittelt:

```java
    public Rahmenwerk getRahmenwerk() {
        final IEclipseContext serviceContext = EclipseContextFactory
                .getServiceContext(getBundle().getBundleContext());
        return serviceContext.get(Rahmenwerk.class);
    } 
```
Zu beachten ist dabei, dass der Zugriff auf die Services auf diesem Weg erst erfolgen kann, 
wenn das Plug-in aktiviert wurde. Insbesondere sollten keine Klassen die �ber ExtensionPoints 
instantiiert werden auf die Services zur�ckgreifen.

### Zugriff �ber Dependency Injection

Der eleganteste Weg w�re der Zugriff auf die Services w�re die Verwendung von Dependency Injection, 
die integraler Bestandteil der E4 Platform ist.

Da jedoch auf Grund des aktuellen Status der E$ Komponenten und Entwicklungswerkzeuge, sowie im 
Hinblick auf die m�glichst hohe Kompatibilit�t der bestehenden BuV-Software-Komponenten, entschieden
wurde die Eclipse RCP Platform im Kompatibilit�tsmodus zu Eclipse 3 zu betreiben, stehen die 
entsprechenden M�glichkeiten nicht vollumf�nglich zur Verf�gung.

Prinzipiell ist es jedoch m�glich neue Plug-ins und Komponenten als Erweiterung des vorhandenen
Applikationsmodell als reine E4 modellbasierte Bestandteile zu intergrieren oder alternativ die
Eclipse3-Bridge aus dem E4-Tools-Projekt einzusetzen, um die M�glichkeiten des Dependency Injection-Mechanismus
zu verwenden.

Entsprechende Literatur und Online-Artikel sind in gro�em Umfang vorhanden, deshalb wird hier 
nicht n�her darauf eingegangen.

Nachstehend sind die grundlegenden vom Rahmenwerk bereitgestellten Services �berblicksm��ig dargestellt.

## Rahmenwerk

Der Service stellt im Wesentlichen die Verbindung zum Datenverteilersystem zur Verf�gung mit dem
die Oberfl�che verbunden werden soll.

Daneben werden einige Informationen zur allgemeinen Anwendungsumgebung geliefert, 
die optional auch im Offline-Betrieb zur Verf�gung stehen. 

```java	
    public interface Rahmenwerk {

        String JOB_FAMILY = "RahmenwerkJobs";

        ArgumentList getStartParameter();
        String getBenutzerName();
        SystemObject getBenutzer();
        
        boolean isOnline();
        ClientDavInterface getDavVerbindung();

        void addDavVerbindungsListener(DavVerbindungsListener listener);
        void removeDavVerbindungsListener(DavVerbindungsListener listener);

        String getPasswort();
        boolean usesBerechtigungenNeu();
        
        SystemObject getOberflaechenObject();
        RwToolBarManager getRwToolBarManager();
    }
```

### isOnline und getDavVerbindung
dient zur �berpr�fung und zum ermitteln der potentiell vorhandenen Datenverteilerverbindung.
Der Service bietet au�erdem die M�glichkeit, sich f�r die Benachrichtigung �ber den Zustandswechsel 
der Datenverteilerverbindung zu registrieren. 

### add/removeDavVerbindungsListener
dient zur Anmeldung eines Listeners, der �ber den Status der Datenverteilerverbindung des
Rahmenwerks informiert wird.

```java
    public interface DavVerbindungsListener {

        /**
        * es wurde eine Datenverteilerverbindung hergestellt.
        * 
        * @param event
        *            die Daten des Ereignisses
        */
        void verbindungHergestellt(DavVerbindungsEvent event);

        /**
        * es wurde eine Datenverteilerverbindung getrennt.
        * 
        * @param event
        *            die Daten des Ereignisses
        */
        void verbindungGetrennt(DavVerbindungsEvent event);

        /**
        * eine Datenverteilerverbindung soll getrennt werden. Der Listener kann
        * hier ein Veto einlegen, sollte aber versuchen einen Zustand zu erreichen,
        * der beim einem der n�chsten Aufrufe kein Veto mehr erfordert.
        * 
        * @param event
        *            die Daten des Ereignisses
        * @return <code>true</code>, wenn die Datenverteilerverbindung offen
        *         gehalten werden soll, <code>false</code> sonst (kein Veto)
        */
        boolean verbindungHalten(DavVerbindungsEvent event);
    }
```

### JOB_FAMILY
ist die ID f�r die Job-Familie, deren Beendigung abgewertet wird, bevor die Rahmenwerk-Applikation 
tats�chlich beendet wird. Damit k�nnen Hintergrundprozesse gegebenenfalls noch anstehende
Arbeiten beenden. 

Um einen Job der Familie zuzuordnen muss die Funktion *belongsTo* implementiert werden, wie 
nachstehend beispielhaft dargestellt ist:

```java
       @Override
       public boolean belongsTo(final Object family) {
            return Rahmenwerk.JOB_FAMILY.equals(family) || super.belongsTo(family);
       }
```

Die Blockierung des Rahmenwerks ist nur bedingt sicher, ein "forced" -Abbruch ist jederzeit m�glich!      

### getXXX-Funktionen
die Funktionen liefern spezielle Informationen zur Rahmenwerk-Applikation:

- die �bergebenen Kommandozeilen-Argumente
- den Name des angeneldeten Benutzers im Online-Betrieb
- das Systemobjekt das angemeldeten Benutzers im Online-Betrieb
- das bei der Herstellung der Datenverteilervebindung eingegebene Passwort
- die Art der verwendeten Berechtigungsklassen
- das Objekt an dem die Parameter f�r die Bedienoberfl�che hinterlegt sind (im Online-Betrieb) 
- den Toolbar-Manager des Hauptfensters der Rahmenwerks-Applikation
	
## Einstellungen
Der Service bietet den Zugriff auf die allgemeinen und benutzerspezifischen Einstellungen 
des Rahmenwerks.

Der Service wird durch eine Instanz der Schnittstelle *Einstellungen* implementiert.

```java
    public interface Einstellungen {

        /**
        * liefert die f�r die �bergebene Adresse vorliegende Einstellung.
        * 
        * @param adresse
        *            die Adresse
        * @return das Einstellungsobjekt
        * @throws IOException
        *             die Einstellung konnte nicht gelesen werden
        */
        Object getValue(final EinstellungsAdresse adresse) throws IOException;

        /**
        * setzt f�r die �bergebene Adresse die angegebene Einstellung.
        * 
        * @param adresse
        *            die Adresse
        * @param einstellung
        *            das Einstellungsobjekt
        * @throws IOException
        *             die Einstellung konnte nicht gelesen werden
        */
        void setValue(final EinstellungsAdresse adresse, final Object einstellung)
                throws IOException;

        /**
        * setzt f�r die �bergebene Adresse die angegebene Einstellung mit den
        * �bergebenen Urlasserinformationen.
        * 
        * @param adresse
        *            die Adresse
        * @param einstellung
        *            das Einstellungsobjekt
        * @param urlasser
        *            die Urlasserinformationen
        * @throws IOException
        *             die Einstellung konnte nicht gelesen werden
        */
        void setValue(final EinstellungsAdresse adresse, final Object einstellung,
                final UrlasserInfo urlasser) throws IOException;

        /**
        * entfernt die unter der �bergebenen Adresse hinterlegten Einstellungen.
        * 
        * @param adresse
        *            die Adresse
        */
        void removeValue(EinstellungsAdresse adresse);

        /**
        * f�gt einen Listener hinzu, der �ber �nderungen von Einstellungen
        * informiert wird.
        * 
        * @param listener
        *            der Listener
        */
        void addEinstellungsListener(final EinstellungChangeListener listener);
    
        /**
        * f�gt einen Listener hinzu, der �ber �nderungen von Einstellungen der
        * angegebenen Kategorie informiert wird.
        * 
        * @param listener
        *            der Listener
        * @param category
        *            die Kategorie
        */
        void addEinstellungsListener(final EinstellungChangeListener listener,
                final String category);

        /**
        * entfernt einen Listener, der �ber �nderungen der Einstellungen informiert
        * wurde.
        * 
        * @param listener
        *            der Listener
        */
        void removeEinstellungsListener(final EinstellungChangeListener listener);

        /**
        * entfernt einen Listener, der �ber �nderungen der Einstellungen in der
        * angegebenen Kategorie informiert wurde.
        * 
        * @param listener
        *            der Listener
        * @param category
        *            die Kategorie
        */
        void removeEinstellungsListener(final EinstellungChangeListener listener,
                final String category);

        /**
        * f�gt einen Listener hinzu, der �ber die Verf�gbarkeit des
        * Einstellungsspeichers informiert wird.
        * 
        * @param listener
        *            der Listener
        */
        void addEinstellungsAvailabilityListener(
                final EinstellungAvailabilityListener listener);

        /**
        * entfernt einen Listener, der �ber die Verf�gbarkeit des
        * Einstellungsspeichers informiert wird.
        * 
        * @param listener
        *            der Listener
        */
        void removeEinstellungsAvailabilityListener(
                final EinstellungAvailabilityListener listener);
    }
```

Eine detaillierte Beschreibung des Zugriffs auf die Einstellungen erfolgt in einem gesonderten Kapitel. 

## Berechtigungen	 
Der Service bietet den Zugriff auf die von den Plug-ins des Rahmenwerks definierten Berechtigungen
mit den in der Regel bestimmte Funktionalit�ten auf Berechtigungsklassen bezogen verf�gbar gemacht oder
entzogen werden k�nnen. 

Der Service wird durch eine Instanz der Schnittstelle *Berechtigungen* implementiert.

```java
    public interface Berechtigungen {

        /**
        * F�gt eine Funktion zur Liste der Funktionen mit Berechtigungen hinzu.
        * 
        * @param funktion
        *            Funktion, die hinzugef�gt werden soll.
        */
        void addOberflaechenFunktion(final FunktionMitBerechtigung funktion);

        /**
         * Entfernt eine Funktion aus der Liste der Funktionen mit Berechtigungen.
        * 
        * @param id
        *            ID der Funktion, die entfernt werden soll.
        */
        void removeOberflaechenFunktion(final String id);

        /**
        * liefert die registrierten Funktionen.
        * 
        * @return die Menge der Funktionen
        */
        Collection<FunktionMitBerechtigung> getFunktionen();

        /**
        * pr�ft, ob der als SystemObject �bergebene Benutzer, die Berechtigung f�r
        * die angegebene Funktion besitzt.
        * 
        * @param benutzer
        *            der Benutzer
        * @param funktion
        *            die Funktion
        * @return der Zustand der Berechtigung
        */
        boolean hasBerechtigung(SystemObject benutzer,
                FunktionMitBerechtigung funktion);

        /**
        * pr�ft, ob der Benutzer mit der �bergebenen PID, die Berechtigung f�r die
        * angegebene Funktion besitzt.
        * 
        * @param benutzerPid
        *            die PID
        * @param funktion
        *            die Funktion
        * @return der Zustand der Berechtigung
        */
        boolean hasBerechtigung(String benutzerPid, FunktionMitBerechtigung funktion);

        /**
        * liefert die Funktion mit der angegebenen Id oder <code>null</code>, wenn
        * keine solche existiert.
        * 
        * @param id
        *            die ID
        * @return die Funktion oder <code>null</code>
        */
        FunktionMitBerechtigung getFunktion(String id);

        /**
        * liefert einen Sammlung der verf�gbaren Berechtigungsklassen (als
        * SystemObject).
        * 
        * @return die Berechtigungsklassen
        */
        Collection<SystemObject> getBerechtigungsKlassen();

        /**
        * f�gt einen Listener hinzu, der �ber �nderungen von Berechtigungen
        * informiert wird.
        * 
        * @param listener
        *            der Listener
        */
        void addOberflaechenFunktionsListener(IBerechtigungListener listener);

        /**
        * f�gt einen Listener hinzu, der �ber �nderungen von Berechtigungen der
        * angegebenen Funktion informiert wird.
        * 
        * @param listener
        *            der Listener
        * @param funktion
        *            die Funktion
        */
        void addListenerForFunktion(IBerechtigungListener listener,
                FunktionMitBerechtigung funktion);

        /**
        * entfernt einen f�r �nderungen von Berechtigungen angemeldeten Listener.
        * 
        * @param listener
        *            der Listener
        */
        void removeListener(IBerechtigungListener listener);
```

Eine detaillierte Beschreibung des Zugriffs auf die Berechtigungen erfolgt in einem gesonderten Kapitel. 
