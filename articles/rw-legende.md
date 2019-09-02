# Legende

## Hintergrund

Die Ansicht "Legende" aus dem ursprünglichen Rahmenwerk wurde durch eine Legende ersetzt,
die als frei schwebendes Legenden-Fenster an jeden beliebigen *WorkbenchPart* gekoppelt 
werden kann.

Die Legende wird über ein vom Rahmenwerk bereitgestelltes Kommando durch den Nutzer
ein- bzw. ausgeblendet.

Damit einem *WorkbenchPart* eine Legende zugeordnet und diese angezeigt werden kann, 
muss dieser die Schnittstelle *ITreeLegende* oder *ICustomLegende* implementieren 
oder einen Adapter für eine dieser Schnittstellen liefern.

Der Inhalt der Legende wird durch die konkrete Implementierung bestimmt.

Liefert der *WorkbenchPart* einen Adapter für die Schnittstelle *IUpdateTimeProvider*,
wird im Legendenfenster eine Aktualisierungszeit eingeblendet, die über die Schnittstelle
per Polling abgerufen wird.

## Schnittstelle ILegende

Die Schnittstelle beschreibt eine für einen WorkbenchPart verfügbare Legende. Sie wird 
durch die konkreteren Schnittstellen *ITreeLegende* oder *ICustomLegende* erweitert.

```java
public interface ILegende {

    /**
     * Die Ecke des Workbench Parts in der das Toolfenster ausgerichtet werden
     * soll.
     */
    static enum Corner {
        /** Oben links. */
        TopLeft("Oben links"),

        /** Oben rechts. */
        TopRight("Oben rechts"),

        /** Unten links. */
        BottomLeft("Unten links"),

        /** Unten rechts. */
        BottomRight("Unten rechts");

        private final String bezeichnung;

        private Corner(final String bezeichnung) {
            this.bezeichnung = bezeichnung;
        }

        public String getBezeichnung() {
            return bezeichnung;
        }
    }

    /**
     * Gibt ein Control zurück, an dem das {@link LegendeWindow} ausgericht
     * wird. Verändert das Control seine Position, dann folgt die Legende dem
     * Control.
     * 
     * <p>
     * Das Control, welches hier zurückgegeben wird, ist <em>nicht</em> das
     * Control der Legende, sondern ein Control im, Workbench Part für das die
     * Legende angezeigt wird.
     * 
     * @return das Control oder <code>null</code>, wenn die Legende nicht folgen
     *         soll.
     */
    Control getControl();

    /**
     * liefert die Ecke des Workbenchparts an dem die anzuzeigende Legende
     * standardßig ausgerichtet werden soll.
     * 
     * @return die Ecke
     */
    Corner getDefaultCorner();

    /**
     * liefert einen Titel zur Anzeige im Legendenfenster (optional).
     * 
     * @return den Titel oder <code>null</code>
     */
    String getTitel();
}
```

## Schnittstelle ICustomLegende

Dies ist die Schnittstelle für eine nicht näher bestimmte nutzerdefinierte Legende.
Der Inhalt der Legende wird über die Funktion *createControl* erzeugt. Es wird 
lediglich das übergeordnete Composite, das befüllt werden soll, übergeben.

```java
public interface ICustomLegende extends ILegende {

    /**
     * erzeugt das anzuzeigende Control.
     * 
     * @param parent
     *            der umgebende Container, in dem das Control eingebettet werden
     *            soll
     */
    void createControl(Composite parent);

}
```

## Schnittstelle ITreeLegende
ist die Schnittstelle für eine Legende, die Elemente in einer Baumansicht darstellt.
Der Inhalt der Legende wird über die Funktion *getBausteine* erzeugt, welche die 
Wurzelelemente der Baumdarstellung liefern. 

```java
public interface ITreeLegende extends ILegende {

    /**
     * Gibt die Bestandteile der Legende zurück.
     * 
     * @return die Liste der Bausteine
     */
    List<ILegendeBaustein> getBausteine();

}

public interface ILegendeBaustein {

    /**
     * Gibt die untergeordneten Legendenbausteine zurück. Gibt es keine, wird
     * eine leere Liste zurückgegeben.
     * 
     * @return die Liste der untergeordneten Bausteine
     */
    List<ILegendeBaustein> getBausteine();

    /**
     * Gibt das kleines Bild des zu beschreibenden Objekts zurück.
     * 
     * @return das Image für die Darstellung des Elements oder <code>null</code>
     */
    Image getIcon();

    /**
     * Gibt eine kurze Objektbeschreibung zurück.
     * 
     * @return die Beschreibung des Elements zur Darstellung im Legendenbaum
     */
    String getText();

}
```

Die Bausteine selbst können wieder untergeordnete Bausteine enthalten, womit sich 
letztendlich eine Baumdarstellung ergibt.

Ein Baustein liefert ein anzuzeigendes Label und ein optionales Icon.



