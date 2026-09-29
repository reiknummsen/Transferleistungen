# TK-Classic-Architektur: Das ViewAgent-Pattern der Testnachrichten-Anwendung

> Autor: Reik Nummsen (P232725, IT.TA.LEVE) · Copilot (2026-09-22)
> Wissensdokument 1 von 7 · Quellpackage: `kur.ui.qs/src/de/tk/ui/viewagent/leistung/kur/datenaustausch/testnachricht`

---

## 1. Das ViewAgent-Pattern in TKeasy

Die Legacy-Anwendung folgt dem TK-eigenen **ViewAgent-Pattern**, einer MVC-Variante mit vier Rollen:

| Rolle | Klasse im Beispiel | Aufgabe |
|---|---|---|
| **Step** | `DatenaustauschRehaStep` | Workflow-Einstieg; liest Anwendungsparameter, lädt BOs, erzeugt den Agent |
| **Agent** | `DatenaustauschRehaAgent` | „Controller/Model“: Geschäftslogik, BO-Erzeugung, XML-Serialisierung, Import |
| **Struct** | `AbstractRootElementStruct`, `AufnahmeTestnachrichtStruct`, … | Datenhalter aus `ViewAgentValue<T>`-Feldern **und** Erzeuger der Swing-Panels |
| **View/Panel** | `DatenaustauschRehaView`, `DatenaustauschRehaPanel` | Swing-Darstellung, Dateidialoge, Tastaturkürzel |

Die Verknüpfung erfolgt datengetrieben über **`ViewAgentValue`**-Objekte: Ein Swing-Widget wird per `setHeldValue(vav)` an ein Value-Objekt gebunden; Änderungen propagieren über `ValueChangeListener`. Das ist bereits ein **Observer-Pattern** und funktional mit dem späteren ereignisbasierten TKX-Datenfluss vergleichbar.

### 1.1 Einstieg über die Anwendungskapsel

```xml
<!-- kur.ui.qs/.../DatenaustauschReha.ac (Auszug, 1 von 11 Blöcken) -->
<application id="AufnahmeNachrichtNeu" mode="insert" type="single" scope="Kur">
    <constant_parameter name="Nachrichtentyp">Aufnahme</constant_parameter>
    <tkeasy_bearbeitung
        workitem="de.tk.ui.viewagent.leistung.kur.datenaustausch.testnachricht.DatenaustauschReha"/>
</application>
```

**Zentrale Beobachtung:** Der Nachrichtentyp wird als **String-Konstante** aus der XML-Konfiguration in den Java-Code durchgereicht. Elf nachrichtentypbezogene Anwendungseinträge zeigen auf **dasselbe** Workitem; die Datei enthält insgesamt 13 `<application>`-Elemente. Die Typunterscheidung muss deshalb zur Laufzeit im Java-Code erfolgen — die Ursache der später beschriebenen `if`-Kaskaden.

```java
// DatenaustauschRehaStep
nachrichtentyp = (String) getParameter("Nachrichtentyp").getContent();
// ...
agent = new DatenaustauschRehaAgent(kur, versicherter, nachrichtentyp);
```

---

## 1.2 Welche Architektur TK Classic nahelegt

TK Classic ist nicht musterlos. Das ViewAgent-Framework stellt mit Agent, View, Struct, Step, `ViewAgentValue` und Listenern bereits Rollen und Mechanismen bereit. Diese Vorgaben fördern insbesondere Observer, Template Method und hierarchische Struct-Komposition. Gleichzeitig orientiert sich die Erweiterung stark an Framework-Vererbung, konkreten Swing-Komponenten und gemeinsamem veränderlichem Zustand.

Für den untersuchten Anwendungsfall hat dies zwei Folgen:

1. **Technische Struktur wird vom Framework vorgegeben:** Neue Funktionen werden typischerweise in Step, Agent, Struct und Panel eingeordnet. Das schafft Konvention, verteilt einen fachlichen Variationspunkt aber über mehrere technische Rollen.
2. **Polymorphie ist an Frameworktypen gebunden:** Der Struct-Baum ist polymorph navigierbar, seine fachlich relevanten Operationen besitzen jedoch keine einheitliche Component-Schnittstelle. Typabhängige Zusammenarbeit wird daher teilweise über Strings, `instanceof` und Casts hergestellt.

TKX unterscheidet sich später nicht dadurch, dass es als erstes Patterns erlaubt. Der Unterschied liegt darin, dass es abstrakte Komponenteninterfaces, Factory-Erzeugung, uniforme Container und schlanke Ereignisschnittstellen bereitstellt. Dadurch lässt sich Variabilität leichter in anwendungseigene Strategy- und Komponentenverträge verlagern, statt sie in Framework-Hooks und Vererbungshierarchien abzubilden.

---

## 2. Die Struct-Hierarchie — ein GoF-nahes strukturelles Composite

```
AbstractVAVStruct                     (Framework)
\-- AbstractElementStruct             getChildrenStructs() + statische Rekursion
    |-- EinfachElementStruct          Blatt (leer!)
    |-- MehrfachElementStruct         Komposit für n-fach wiederholbare Elemente
    \-- AbstractRootElementStruct<T>  Wurzel: Datei-I/O, XML-Bytes, init/update/createPanel
        \-- AbstractTestnachrichtStruct<T>   Kopfdaten + Admindaten + Dokumente
            |-- AufnahmeTestnachrichtStruct
            |-- EntlassungTestnachrichtStruct
            |-- RechnungTestnachrichtStruct
            \-- weitere Nachrichtentypen
```

```java
// AbstractElementStruct — Composite-Wurzel mit externer Traversierung
public abstract class AbstractElementStruct extends AbstractVAVStruct {

    public abstract AbstractElementStruct[] getChildrenStructs();

    public static void getChildrenStructRekursiv(List<AbstractElementStruct> result,
                                                 AbstractElementStruct root) {
        for (AbstractElementStruct struct : root.getChildrenStructs()) {
            result.add(struct);
            getChildrenStructRekursiv(result, struct);
        }
    }
}
```

**Befund:** Das Composite ist vorhanden, sein gemeinsamer Operationsvertrag ist jedoch **schmal**. Die Schnittstelle bietet nur `getChildrenStructs()`. Die eigentlichen Operationen (`init`, `update`, `createPanel`) sind **nicht** Teil des Composite-Vertrags, sondern liegen typspezifisch und unterschiedlich signiert auf den Unterklassen:

```java
// AbstractRootElementStruct<T extends KurNachrichtImpl>
public abstract void init(T kurNachricht);
public abstract void update(T kurNachricht);
public abstract void createPanel(TkPanel main, JTabbedPane tabbedPane);
```

```java
// AufnahmeTestnachrichtStruct — Kinder werden manuell aufgezählt
@Override
public AbstractElementStruct[] getChildrenStructs() {
    return new AbstractElementStruct[] {
        kindMutterKindMassnahmen, getKopfdaten(), kommunikation, rehabilitand,
        lebendspender, aufnahmediagnosen, getAdmindaten(), getDokumente()
    };
}
```

Da `begleitpersonen` in dieser Liste **fehlt**, ist die manuell gepflegte Kindliste strukturell inkonsistent und birgt das Risiko, dass baumbasierte Querschnittsoperationen diesen Teilbaum nicht erreichen. Ein konkreter Laufzeit- oder Validierungsfehler ist daraus allein jedoch nicht nachgewiesen, weil Framework-Validierung zusätzlich über den internen VAV-Baum erfolgen kann.

Die Traversierung wird ausschließlich für **Querschnittsaufgaben** genutzt, per `instanceof`-Abfrage statt polymorph:

```java
// DatenaustauschRehaAgent#initFromVersicherter — Traversierung + instanceof statt Polymorphie
List<AbstractElementStruct> result = new ArrayList<AbstractElementStruct>();
AbstractElementStruct.getChildrenStructRekursiv(result, struct);
for (AbstractElementStruct struct : result) {
    if (struct instanceof DatenuebernahmeVersicherterStruct) {
        ((DatenuebernahmeVersicherterStruct) struct).initVersicherter(versicherter, lebendspende);
    }
}
```

`DatenuebernahmeVersicherterStruct` und `DatenuebernahmeKurStruct` sind dabei **Rollen-Interfaces** mit den Operationen `initVersicherter(...)` beziehungsweise `initKur(...)`. Die Traversierung mit anschließendem `instanceof`-Dispatch ist kein Visitor-Pattern: Ein Visitor und eine einheitliche `accept`-Operation fehlen.

---

## 3. Der Agent als God Class

`DatenaustauschRehaAgent` umfasst **623 LOC** und vereint mindestens sechs Verantwortlichkeiten:

1. BO-Erzeugung je Nachrichtentyp (`fromXML`)
2. Fachliche Vorbelegung von Kopf-/Admindaten (`doVorbelegung`)
3. Auswahl der Struct-Klasse (`getStructClass`)
4. Delegation von `init`/`update` an das konkrete Struct (`callInit`, `callUpdate`)
5. XML-Serialisierung/Deserialisierung (`toXML`, `toXML2`)
6. Import in die Verarbeitungsstrecke inkl. EAI-Service-Lookup (`update`, `starteFolgeEAI`)

### 3.1 Die fünffache `if`-Kaskade

Der gleiche Variationspunkt wird **fünfmal** in leicht abgewandelter Form ausgewertet:

```java
// (1) Struct-Klasse wählen
@Override public Class<?> getStructClass() {
    if (isAufnahme())           return AufnahmeTestnachrichtStruct.class;
    if (isAbsageEinrichtung())  return AbsageEinrichtungTestnachrichtStruct.class;
    /* … 9 weitere … */
    throw new AgentCreateException(nachrichtentyp + " ist unbekannter Nachrichtentyp");
}

// (2) BO erzeugen
if (isAufnahme())                kurNachricht = KurNachrichtErzeuger.erzeugeAufnahme();
else if (isAbsageEinrichtung())  kurNachricht = KurNachrichtErzeuger.erzeugeAbsageEinrichtung();
/* … */

// (3) Default-Version setzen
if (isAufnahme())                defaultVersion = Version.AUFNAHME;
else if (isAbsageEinrichtung())  defaultVersion = Version.ABSAGE_EINRICHTUNG;
/* … */

// (4) init mit Doppel-Downcast
if (isAufnahme()) {
    ((AufnahmeTestnachrichtStruct) struct).init((KurAufnahmeImpl) kurNachricht);
    return;
}

// (5) update wiederholt denselben Dispatch mit Doppel-Downcast
if (isAufnahme()) {
    ((AufnahmeTestnachrichtStruct) struct).update((KurAufnahmeImpl) kurNachricht);
    return;
}
```

Der Typvergleich erfolgt über **String-Literale**, die mit den `.ac`-Konstanten übereinstimmen müssen — inklusive Umlaute und Sonderzeichen:

```java
private boolean isAntragAufVerlaengerung() { return nachrichtentyp.equals("Antrag auf Verlängerung"); }
private boolean isZuzahlungsnachricht()    { return nachrichtentyp.equals("Zuzahlungsgutschrift/-rückforderung"); }
```

### 3.2 Quantifizierung des Änderungsaufwands

Ein **neuer Nachrichtentyp** erfordert im Legacy-Stand Änderungen an mindestens:

| # | Ort | Art |
|---|---|---|
| 1 | `DatenaustauschReha.ac` | neuer `<application>`-Block + Alias |
| 2 | `DatenaustauschRehaAgent#getStructClass` | neuer `if`-Zweig |
| 3 | `DatenaustauschRehaAgent#fromXML` | neuer `else if`-Zweig |
| 4 | `DatenaustauschRehaAgent#doVorbelegung` | neuer `else if`-Zweig |
| 5 | `DatenaustauschRehaAgent#callInit` | neuer `if`-Zweig mit Doppelcast |
| 6 | `DatenaustauschRehaAgent#callUpdate` | neuer `if`-Zweig mit Doppelcast |
| 7 | `DatenaustauschRehaAgent` | neue `isXyz()`-Prädikatmethode |
| 8 | neue `XyzTestnachrichtStruct` | Neuanlage (200–400 LOC) |

-> **7 Änderungen an bestehendem Code**, davon 6 in einer einzigen Klasse. Klare Verletzung des **Open-Closed-Principle**.

---

## 4. Verschränkung von Daten, Logik und Darstellung

Das Struct ist gleichzeitig Datencontainer **und** Layout-Code. `AufnahmeTestnachrichtStruct` enthält neben `init`/`update` auch:

```java
@Override
protected TkPanel createFachdatenPanel(TkPanel main) {
    final TkPanel verwaltungsdatenPanel = new TkPanel();
    verwaltungsdatenPanel.setLayout(new ProportionLayout(false, true));
    verwaltungsdatenPanel.setBorder(TkGuiStandards.CONTENT_BORDER);

    final TkPanel panelAufnahmedatenZusatz = new TkPanel();
    createPanelAufnahmedatenZusatz(panelAufnahmedatenZusatz);   // ~80 LOC Layoutcode
    verwaltungsdatenPanel.add(panelAufnahmedatenZusatz);

    final TkPanel panelRehabiltand = new TkPanel();
    getRehabiltand().createPanel(main, panelRehabiltand);
    verwaltungsdatenPanel.add(panelRehabiltand);
    /* … 5 weitere Blöcke, jeweils 2–4 Zeilen Panel-Boilerplate … */
    return verwaltungsdatenPanel;
}
```

Das manuelle Layout erfolgt über **Guide-Lines**, d. h. explizite Zeilenobjekte:

```java
final TkGuideLineLayout layout = new TkGuideLineLayout(0);
final TkGuideLine label1  = layout.addNewLine();
final TkGuideLine felder1 = layout.addNewLine();
final TkGuideLine label2  = layout.addNewLine();
final TkGuideLine felder2 = layout.addNewLine();
pC.setLayout(layout);

final TkComboBox<Behandlungsart> behandlungsart =
    new TkComboBox<Behandlungsart>(Behandlungsart.pick().getGueltigeList());
behandlungsart.setHeldValue(this.behandlungsart);   // Observer-Bindung
behandlungsart.setHelpID("Behandlungsart");
behandlungsart.setVisualizerName("Behandlungsart");
behandlungsart.setFieldMandatory(true);
pC.add(LabelFactory.createFieldLabel("Behandlungsart"), label2);
pC.add(behandlungsart, new TkGuideLineConstraint(felder2, false));
```

Pro Eingabefeld: ~7 Zeilen, verteilt auf Erzeugung, Bindung, Metadaten, Label-Platzierung und Constraint. **Die Zuordnung Label/Feld ist rein positionell** und damit im Code nicht lokal nachvollziehbar.

Zusätzlich existiert eine dritte Bauebene: `AbstractTestnachrichtStruct#createPanel` erzeugt die Tabs direkt inklusive Mnemonics:

```java
tabbedPane.addTab("[1] Fachdaten", new TkEditScrollPane(createFachdatenPanel(main)));
tabbedPane.setMnemonicAt(0, KeyEvent.VK_1);
tabbedPane.addTab("[2] Administration", new TkEditScrollPane(administrationsdatenPanel));
tabbedPane.setMnemonicAt(1, KeyEvent.VK_2);
```

---

## 5. Strukturelle Grenzen und Trade-offs

Die folgenden Befunde sind keine pauschale Abwertung von TK Classic. Das Framework erfüllt mit ViewAgent, Structs und Value-Bindung zentrale Anforderungen seiner Entstehungszeit und stellt selbst mehrere wiederverwendbare Muster bereit. Die Tabelle zeigt vielmehr, warum die konkrete Anwendung mit wachsender Zahl an Nachrichtentypen schwerer zu erweitern und isoliert zu testen ist.

| Symptom | Fundstelle | Wirkung |
|---|---|---|
| **God Class** | `DatenaustauschRehaAgent` (623 LOC, 6 Verantwortlichkeiten) | SRP-Verletzung, hohe Änderungsrate |
| **Shotgun Surgery** | 7 Codestellen je neuem Nachrichtentyp | Fehleranfälligkeit, hoher Aufwand |
| **Stringly typed** | `nachrichtentyp.equals("Antrag auf Verlängerung")` | Tippfehler erst zur Laufzeit sichtbar |
| **Doppel-Downcast** | `((AufnahmeTestnachrichtStruct) struct).init((KurAufnahmeImpl) kurNachricht)` | Typsicherheit umgangen, `ClassCastException` möglich |
| **`instanceof`-Dispatch** | `initFromVersicherter`, `initFromKur` | Polymorphie verschenkt |
| **Inkonsistentes Composite** | `begleitpersonen` fehlt in `getChildrenStructs()` | Risiko unvollständiger baumbasierter Querschnittsoperationen; konkreter Fehler nicht belegt |
| **UI im Datenmodell** | `createFachdatenPanel` im Struct | erhöhte Testkopplung und zusätzlicher Swing-/Headless-Aufwand |
| **Zeichenweises String-Handling** | `transferContent`: `for (i…) content.addString(s.substring(i,i+1))` | O(n) Value-Objekte je XML-Datei; mögliches Performancerisiko |
| **`System.err` / `e.printStackTrace()`** | `initStruct`, `writeTempFile` | kein strukturiertes Logging |
| **Statischer Service-Lookup** | `ESServices.locateService(...)`, `Injector.injector().get(...)` | Testisolation nur mit Framework-Extension möglich |
| **Toter/auskommentierter Code** | `toXML2`, `// ViewAgentValue<String> zielDateiString` | Verständnisaufwand |
| **Kommentare als Entscheidungsprotokoll** | „ULLA: BOs erzeugen, nur wenn…“, „André: Nach Rücksprache…“ | Wissen nicht im Code verankert |

### 5.1 Kopplungsanalyse

`DatenaustauschRehaAgent` importiert **44 Typen**, davon 30 aus `de.tk.biz.…` (Domäne/Persistenz), u. a. `KurAufnahmeImpl`, `KurNachrichtImport`, `RehaNachrichtenXMLService`, `KurNachrichtPrueferES`, `GlobalEnvironment`.

-> Eine UI-Klasse hängt direkt an **Implementierungsklassen** (`…Impl`) der Domäne und an Infrastruktur (EAI, XML, Dateisystem). Dies steht in Spannung zur dokumentierten hexagonalen Zielarchitektur des Projekts, nach der Adapter über Ports mit der Domäne kommunizieren sollen.

---

## 6. Zwischenfazit

Der Legacy-Code ist **nicht musterlos**: Observer (VAV-Bindung), Composite (Struct-Baum), Template Method (`AbstractTestnachrichtStruct`) und ein Framework-Fabrik-Hook (`getStructClass`) sind vorhanden. Ihre Wirkung auf den zentralen Variationspunkt bleibt jedoch begrenzt, weil

- das Composite keinen **gemeinsamen Operationsvertrag** trägt,
- der Fabrik-Hook nur den **Struct-Typ** liefert, während weiteres typabhängiges Verhalten auf vier zusätzlichen Kaskaden verteilt bleibt, und
- der Variationspunkt „Nachrichtentyp“ nicht als **Objekt**, sondern als **String** modelliert ist.

Die TKX-Zielarchitektur adressiert diese Punkte durch einen stärkeren Abschnittsvertrag, eine zentrale Registry und Strategy-Objekte (siehe Dokument 02 und 03). Ein String bleibt als Auswahlschlüssel erhalten; das zugehörige Verhalten ist jedoch nicht mehr über mehrere String-Abfragen verteilt, sondern in Konfigurationsobjekten gekapselt.

Observer, Composite und Template Method leisten im Legacy-Code reale Wiederverwendung. Die zentrale Schwäche besteht nicht im Fehlen von Entwurfsmustern, sondern darin, dass der fachliche Variationspunkt „Nachrichtentyp“ nicht mit diesen Mechanismen gekapselt ist. TKX stellt dafür günstigere Komponenten- und Ereignisschnittstellen bereit; der architektonische Fortschritt entsteht jedoch erst durch die bewusste Kombination dieser Frameworkmöglichkeiten mit Strategy und modularen Abschnitten.
