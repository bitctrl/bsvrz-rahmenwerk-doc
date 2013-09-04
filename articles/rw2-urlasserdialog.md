Title:   Rahmenwerk 2.0 - Urlasserdialog  
Author:  U. Peuker  
Date:    August 30, 2013  
Comment: Einbindung und Verwendung des Urlasserdialogs im Rahmenwerk 2.0

Urlasserdialog
==============

## Hintergrund

Der Urlasserdialog dient dazu ausgewählte Aktionen mit Anmeldedaten eines Nutzers zu
verififizieren. Der Nutzer der über den Urlasserdialog authentifiziert wird, muss nicht 
der aktuell angemeldete Nutzer sein. Gegebenenfalls wird eine neue Datenverteilerverbindung 
für den abweichenden Nutzer erzeugt, über die dann die mit dem Urlasserdialog verknüpften 
Operationen ausgeführt werden, damit das Berchtigungskonzept des Datenverteilers an
dieser Stelle greifen kann.

## Programmierschnittstelle

### Konstruktor

Der Urlasserdialog steht als JFace-Dialog-Klasse in der Basis-Bibliothek des Rahmenwerks
zur Verfügung.

```java
public UrlasserDatenDialog(final Shell parent,
            final UrlasserDatenSender datenSender) {
        super(parent);
        this.datenSender = datenSender;
}
```

Der Dialog wird mit der *open*-Funktion eines Dialogs geöffnet und liefert die gängigen 
Werte *Window.OK* oder *Window.CANCEL* zurück, je nachdem wie der Dialog geschlossen 
wurde.

Die eingegebenen Urlasserinformationen können anschließend bei Bedarf mit der Funktion

```java
public UrlasserInfo getUrlasserInfo();
```

bestimmt werden. Die potentiell erzeugte neue Urlasserverbindung steht nach dem Abschluss
des Dialogs nicht mehr zur Verfügung. Alle Operationen, die über diese Verbindung ausgeführt
werden sollen, müssen von der im Konstruktor übergebenen Instanz des *UrlasserDatenSender*
übernommen werden.

### UrlasserDatenSender
Eine Instanz dieser Klasse übernimmt die mit einem Urlasserdialog potentiell erzeugte 
oder die aktuelle Datenverteilerverbindung und führt die mit dem Urlasserdialog 
verbundenen Operationen in Bezug zum Datenverteiler aus. Die Ausführung der Operationen
blockiert den Dialog, d.h. er wird erst nach Abschluß aller Operationen des Datensenders 
geschlossen und die Verbindung wird abgebaut.

Die übergebene Klasse muss eine Instanz der folgende Schnittstelle sein:

```java
public interface UrlasserDatenSender {

    /**
     * Callback-Funktion wird aufgerufen, wenn ein Urlasser über den
     * Urlasserdialog identifiziert wurde und optional eine entsprechende
     * Datenverteilerverbindung erstellt wurde.
     * 
     * Nach der Ausführung der gewünschten Operationen wird die Verbindung
     * automatisch wieder geschlossen.
     * 
     * @param verbindung
     *            die Verbindung zum Datenverteiler
     * @param urlasser
     *            die Urlasserinformationen
     */
    void execute(ClientDavInterface verbindung, UrlasserInfo urlasser);
}
```

### DefaultDatenSender

Das Rahmenwerk stellt eine Standard-Implementierung des *UrlasserDatenSender* zur 
Verfügung, mit dem das bisherige Verhalten des Urlasserdialogs (versenden einer
Menge von *ResultData* - Instanzen) nachgebildet wurde.

```java
public DefaultDatenSender(final Collection<ResultData> resultDatas);
```

Der Konstruktor übernimmt die Daten, die bei erfolgreicher Authentifizierung
über den Urlasserdialog an den Datenverteiler versendet werden.
       