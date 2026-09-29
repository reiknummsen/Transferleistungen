# TKX-Zielarchitektur: Framework-Fit und musterorientierte UI-Struktur

> Autor: Reik Nummsen (P232725, IT.TA.LEVE) · Copilot (2026-09-22)
> Wissensdokument 2 von 7 · Quellpackage: `testdaten-kur.ui/src/de/tk/ui/tkx/testdaten/kur/davr/testandwendung`

---

## 1. Was ist TKX?

**TKX** ist eine TK-Eigenentwicklung für Benutzungsoberflächen. Die Anwendung beschreibt abstrakte UI-Elemente serverseitig in Java; TKX transportiert Zustand und Ereignisse und stellt die Oberfläche im Browser beziehungsweise Electron-Client mit Web-Technologien dar. Charakteristika, die für die Musterdiskussion relevant sind:

| Eigenschaft | Konsequenz für den Entwurf |
|---|---|
| **Programmatische UI-Komposition mit deklarativen Metadaten** über eine statische `UIFactory` (`newTextInput`, `newGrid`, `newExpandablePanel`, `newTab`) | Weniger manueller Layoutcode; ein Feld entsteht typischerweise in weniger Zeilen |
| **Automatisches Layout** über `Grid` + `GridConstraints` (`SPAN_HALF`, `SPAN_QUARTER`) | Keine Guide-Lines, keine positionelle Label-Zuordnung |
| **Fluent Interface / Method Chaining** (`setName(...).setMandatory(true).addValidator(...)`) | Kompakte, lesbare Bauvorschriften; *Builder*-ähnliche Notation |
| **Lambda-basierte Listener** (`addValueChangeListener`, `addActivationListener`) | Observer ohne anonyme innere Klassen |
| **Komponenten-Container mit `add`/`removeAll`** (`Tab`, `ExpandablePanel`, `Formular`) | Container-Uniformität als tragfähiges Fundament für Komposition |
| **Deklarative Validierung** (`addValidator`, `ValidationResult`, `isValid()`) | Validierung am Feld statt in Agent-Kaskaden |
| **`ComponentId`** als expliziter Komponentenbezeichner (`of(...)`, `named(...)`) | Stabile Identität für automatisierte GUI-Tests |
| **`ApplicationUIConstructor`** als einziger Einstiegspunkt | Definierter *Composition Root* für Dependency Injection |
| **Vorgefertigte Anwendungstypen** (`SinglePageEditApplicationUI`) | Rahmenlogik (OK/Abbrechen, Titel) wird gestellt, nicht gebaut |

Quelle für das serverseitige UI-Modell, die Factory-Erzeugung und die Testbarkeitsziele: interne TKX-Dokumentation `.github/skills/tkx/02-how-tos-grundlagen/technische-grundlagen.md`. Die konkrete Verwendung der Komponenten ist zusätzlich durch die nachfolgend zitierten Klassen belegt.

### 1.1 TKX als Enabler und Leitplanke

TKX schreibt der Anwendung kein Strategy-, Composite- oder MVP-Pattern zwingend vor. Seine API verschiebt jedoch die bevorzugte Gestaltungsrichtung:

- Komponenten werden über `UIFactory` erzeugt und als Interfaces verwendet. Dadurch wird Komposition gegenüber der Ableitung konkreter Widgetklassen begünstigt.
- Container besitzen uniforme `add`- und `removeAll`-Operationen. Austauschbare Abschnittsobjekte können deshalb ohne Kenntnis konkreter Widgets orchestriert werden.
- Komponentenereignisse sind über Listener und Lambdas zugänglich. Observer-basierter Datenfluss ist damit der natürliche Interaktionsmechanismus.
- `ApplicationUIConstructor` bildet einen klaren Composition Root. Presenter, Strategies und weitere Abhängigkeiten können an einer definierten Stelle zusammengesetzt werden.
- Vorgegebene Anwendungstypen, Layouts und Komponenten reduzieren technische Freiheitsgrade. Diese Einschränkung wirkt als Leitplanke: Die Entwicklung konzentriert sich stärker auf fachliche Zerlegung und weniger auf individuelle Swing-Vererbung oder pixelnahe Layoutlogik.
- `ComponentId`, feldnahe Validierung und abstrakte UI-Elemente schaffen stabile Ansatzpunkte für Tests und konsistente Bedienkonzepte.

Der entscheidende Unterschied zu TK Classic lautet daher nicht „Patterns sind erst mit TKX möglich“. TK Classic enthält selbst Observer, Composite und Template Method. TKX macht es jedoch einfacher, **anwendungseigene fachliche Variationspunkte** durch kleine Interfaces und Objektkomposition zu modellieren, weil die Anwendung nicht gleichzeitig konkrete Swing-Widgets, Client-View und ViewAgent-Struct-Hierarchien beherrschen muss.

### 1.2 Einstieg über die Anwendungskapsel

```xml
<!-- testdaten-kur.ui/.../DatenaustauschReha.ac -->
<application id="AufnahmeNachrichtNeu2" mode="insert" scope="Kur" type="single">
    <title>Reha-Test-Nachricht Aufnahme 2</title>
    <optional_parameter name="Versicherter"/>
    <optional_parameter name="LeistGEVOId"/>
    <constant_parameter name="Nachrichtentyp">Aufnahme</constant_parameter>
    <tkx view="de.tk.ui.tkx.testdaten.kur.davr.testandwendung.TestnachrichtUIConstructor"/>
</application>
```

Zwei Beobachtungen für die Arbeit:

1. **Parallele Einführung:** Die neue Kapsel heißt `AufnahmeNachrichtNeu2` und existiert neben `AufnahmeNachrichtNeu`. Diese Koexistenz bildet den Migrationskontext, steht aber nicht im Zentrum der Patternanalyse.
2. **Reduktion des Deskriptors:** Die Legacy-`.ac` benötigt elf nachrichtentypbezogene `<application>`-Blöcke. Die neue Datei enthält einen Anwendungseintrag mit dem konstanten Startparameter `Aufnahme`; die auswählbaren Formvarianten werden zusätzlich aus der Registry in die Combobox geladen. Damit ist die konkrete Formularzusammensetzung nicht mehr ausschließlich an getrennte Deskriptorblöcke gebunden, sondern kann zur Laufzeit gewechselt werden.

---

## 2. Klassenlandkarte

```
de.tk.ui.tkx.testdaten.kur.davr.testandwendung
|-- TestnachrichtUIConstructor        -> TKX-Einstieg und Composition Root
|-- TestnachrichtApplicationUI        -> View-Orchestrierung und Context für Strategy/Abschnitte
|-- TestnachrichtPresenter            -> Presenter-Vertrag der Zielarchitektur
|-- TestnachrichtPresenterImpl        -> Koordination von UI-DTOs und Anwendungslogik
|-- TestnachrichtDTO                  -> Wurzel-DTO (Fallnummer + Admin + Fach)
|-- AdmindatenDTO / FachdatenDTO      -> Datencontainer
|-- StartoperationenPanel             -> Typauswahl, Fallnummer, Dateioperationen
|-- DatenPanel                        -> TabPanel: Fachdaten + Administration
|-- AdministrationPanel               -> Kopf-/Admindaten und Dokumente
|-- TestnachrichtStrategy             -> ungenutzte alternative/frühere Abstraktion
|-- konfiguration/                    -> Strategy-Familie
|   |-- TestnachrichtKonfiguration            (Strategy-Interface)
|   |-- TestnachrichtKonfigurationFactory     (Registry mit factory-artigem Lookup)
|   \-- konkrete Nachrichtentyp-Konfigurationen
|-- abschnitt/                        -> polymorph orchestrierte UI-Bausteine
|   |-- TestnachrichtAbschnitt                (gemeinsamer Komponentenvertrag)
|   |-- einfache Fachabschnitte
|   \-- wiederholbare Mehrfachabschnitte
\-- mehrfach/dto/                     -> DTOs der Mehrfachabschnitte
```

---

## 3. Die vier Bausteine im Detail

### 3.1 Einstieg: `TestnachrichtUIConstructor` (22 LOC)

```java
public class TestnachrichtUIConstructor implements ApplicationUIConstructor {

    @Override
    public ApplicationUI create(Map<String, Object> parameter) {
        TestnachrichtPresenterImpl presenter = new TestnachrichtPresenterImpl(parameter);
        return new TestnachrichtApplicationUI(presenter).getApp();
    }
}
```

Vergleich: Der Legacy-Einstieg besteht aus `DatenaustauschRehaStep` (78 LOC) + `DatenaustauschRehaView` (26 LOC) + `DatenaustauschRehaAgent#init`. Hier: **eine Methode, drei Zeilen**, mit sichtbarer Abhängigkeitsverdrahtung (Composition Root).

### 3.2 Orchestrierung: `TestnachrichtApplicationUI` (85 LOC)

```java
public class TestnachrichtApplicationUI {

    private final SinglePageEditApplicationUI app;
    private final TestnachrichtPresenterImpl presenter;
    private DatenPanel datenPanel;
    private List<TestnachrichtAbschnitt> aktiveAbschnitte;

    public TestnachrichtApplicationUI(TestnachrichtPresenterImpl presenter) {
        this.presenter = presenter;
        app = ApplicationUIFactory.newSinglePageEditApplication(() -> OKResult.OK);
        app.setApplicationTitle("DA Reha - Testnachricht");
        initLayout();
    }

    private void initLayout() {
        TestnachrichtDTO dto = presenter.getDTO();
        datenPanel = new DatenPanel(dto.getAdminDatenDTO(), dto.getFachdatenDTO());
        Tab fachdatenTab = datenPanel.getFachdatenTab();

        StartoperationenPanel startPanel = new StartoperationenPanel(
            presenter,
            List.of(TestnachrichtKonfigurationFactory.getVerfuegbareNachrichtentypen()),
            typ -> onNachrichtentypChanged(typ, fachdatenTab),   // Callback statt if-Kaskade
            this::belegeAlleFelder
        );

        app.getRootContainer().add(startPanel.getPanel());
        app.getRootContainer().add(datenPanel.getPanel());

        String[] nachrichtentypen = TestnachrichtKonfigurationFactory.getVerfuegbareNachrichtentypen();
        if (nachrichtentypen.length > 0) {
            onNachrichtentypChanged(nachrichtentypen[0], fachdatenTab);
        }
    }

    /** Kern der Migration: Dynamischer Formularaufbau ohne jede Typabfrage. */
    private void onNachrichtentypChanged(String label, Tab fachdatenTab) {
        TestnachrichtKonfiguration konfiguration = TestnachrichtKonfigurationFactory.getKonfiguration(label);
        aktiveAbschnitte = konfiguration.getAbschnitte();       // STRATEGY
        fachdatenTab.removeAll();
        FachdatenDTO fachdaten = presenter.getDTO().getFachdatenDTO();
        for (TestnachrichtAbschnitt abschnitt : aktiveAbschnitte) {
            fachdatenTab.add(abschnitt.erstellePanel(fachdaten));   // uniforme Komposition
        }
    }

    private void belegeAlleFelder() {
        TestnachrichtDTO dto = presenter.getDTO();
        datenPanel.getAdministrationPanel().belegeFelder(dto.getAdminDatenDTO(), dto.getFachdatenDTO());
        if (aktiveAbschnitte != null && dto.getFachdatenDTO() != null) {
            for (TestnachrichtAbschnitt abschnitt : aktiveAbschnitte) {
                abschnitt.belegeFelder(dto.getFachdatenDTO());      // uniforme polymorphe Operation
            }
        }
    }
}
```

**Das ist die zentrale Vorher-Nachher-Gegenüberstellung der Arbeit:**
Für die **Formularauswahl und -komposition** wird der zuvor mehrfach ausgewertete Variationspunkt „Nachrichtentyp“ durch **ein Registry-Lookup und uniforme Schleifen** gekapselt. Die Orchestrierung benötigt weder konkrete Nachrichtentypen noch entsprechende Casts. Dieselbe Entwurfslogik lässt sich in einer vollständigen Zielarchitektur auch auf Mapping und Anwendungsfälle übertragen: Typabhängiges Verhalten wird jeweils hinter einem passenden Vertrag gebündelt, statt in zentralen Fallunterscheidungen verteilt zu werden.

> **Pattern-Präzisierung:** `TestnachrichtAbschnitt` bietet einen einheitlichen Operationsvertrag, aber keine Kinderoperation und kein Abschnitt hält eine Liste weiterer `TestnachrichtAbschnitt`-Objekte. Auf Anwendungsebene liegt daher keine vollständige GoF-Composite-Struktur vor, sondern eine **polymorphe Abschnittskomposition**. Für das Formular ist diese flache Struktur angemessen: Beliebig tiefe Traversierung und transparente Gleichbehandlung verschachtelter Composite-Knoten werden fachlich nicht benötigt. Ein vollständiges GoF-Composite würde hier zusätzliche Komplexität erzeugen, ohne einen erkennbaren Mehrwert zu liefern. Composite-artiges Containerverhalten stellt TKX bereits bereit.

### 3.3 Statischer Aufbau: `DatenPanel` (51 LOC)

```java
public DatenPanel(AdmindatenDTO admindaten, FachdatenDTO fachdaten) {
    fachdatenTab = UIFactory.newTab(ComponentId.of("FachdatenTab"));
    fachdatenTab.setTitle("Fachdaten");
    administrationPanel = new AdministrationPanel(admindaten, fachdaten);

    Formular formular = UIFactory.newFormular(ComponentId.of("DatenFormular"));
    TabPanel tabPanel = UIFactory.newTabPanel(ComponentId.of("DatenTabPanel"));
    tabPanel.add(fachdatenTab);
    tabPanel.add(administrationPanel.getTab());
    formular.add(tabPanel);

    panel = UIFactory.newExpandablePanel(ComponentId.of("DatenPanel"));
    panel.setTitle("Daten").setExpandable(false);
    panel.add(formular);
}
```

Der **Administration-Tab ist invariant** über alle Nachrichtentypen, der **Fachdaten-Tab ist variant**. Diese Trennung ist explizit im Code sichtbar: `getFachdatenTab()` wird als Zielcontainer nach außen gereicht und bei jedem Typwechsel geleert und neu befüllt. Im Legacy-Code war dieselbe Unterscheidung nur implizit in `AbstractTestnachrichtStruct#createPanel` und der abstrakten `createFachdatenPanel`-Methode vorhanden (Template Method).

### 3.4 Datenerfassung: `StartoperationenPanel` (123 LOC)

```java
private Formular createAuswahlFormular() {
    Formular formular = UIFactory.newFormular(ComponentId.of("AuswahlFormular"));
    formular.setTitle("Testnachricht-Auswahl");

    Combobox<String> combobox = UIFactory.newCombobox(ComponentId.named("Nachrichtentyp"));
    combobox.setName("Nachrichtentyp");
    combobox.setOptions(new ArrayList<>(nachrichtentypen));   // Optionen aus der Factory
    combobox.addValueChangeListener(event -> {
        String gewaehlterTyp = event.getSource().getValue();
        if (gewaehlterTyp != null) {
            onNachrichtentypGewaehlt.accept(gewaehlterTyp);   // Strategy-Wechsel zur Laufzeit
        }
    });
    formular.add(combobox);
    return formular;
}
```

Deklarative Validierung direkt am Feld — im Legacy-Code lag Validierung verstreut in `DatenaustauschRehaUtil.createStringField(..., regex, hinweis)` und in `struct.isDataValid() && struct.areChildrenDataValid()`:

```java
fallnummer = UIFactory.newTextInput(ComponentId.of("Fallnummer"))
    .setName("Fallnummer")
    .setMandatory(true)
    .addValidator(v -> v.getValue().length() != 13
        ? ValidationResult.invalid("Die Fallnummer muss 13 Zeichen lang sein.")
        : ValidationResult.valid());
```

Die Entkopplung zum Rest der Anwendung erfolgt über **Funktionsschnittstellen** statt Vererbung oder Framework-Callbacks:

```java
public StartoperationenPanel(
    TestnachrichtPresenterImpl presenter,
    Collection<String> nachrichtentypen,
    Consumer<String> onNachrichtentypGewahlt,   // „was passiert bei Typwechsel“
    Runnable onFelderBelegen                    // „was passiert nach Vorbelegung“
)
```

Das ist ein *Command*/*Callback*-Idiom: Das Panel kennt weder `TestnachrichtKonfigurationFactory` noch `TestnachrichtAbschnitt`.

---

## 4. Vergleich zentraler Metriken

| Kriterium | Legacy | TKX |
|---|---|---|
| Java-Dateien im Package(-baum) | 59 | 65 |
| Java-LOC im untersuchten Stand | ca. 8.900 | ca. 5.000 |
| Größte Klasse | `DatenaustauschRehaAgent` 623 LOC | `AdministrationPanel` 327 LOC |
| Einstiegsschicht | Step 78 + View 26 + Agent-`init` | Constructor 22 |
| Typabhängige `if`-Kaskaden | 5 (à bis zu 11 Zweige) | 0 in der TKX-Formularorchestrierung |
| Downcasts auf Nachrichtentyp | 22 (`callInit` + `callUpdate`) | 0 |
| `.ac`-Deskriptorzeilen | 285 (11 Applications) | 34 (1 Application) |
| `de.tk.biz.*`-Importe in der zentralen Orchestrierungsklasse | 30 im Legacy-Agent | 0 in `TestnachrichtApplicationUI`; im übrigen TKX-Package bestehen direkte Business-Typ-Abhängigkeiten |
| LOC je Eingabefeld (typisch) | ~7 | ~3–4 |

> **Methodischer Hinweis für die TFL:** LOC sind nur ein ergänzender Indikator. Belastbarer sind die Strukturmerkmale des vergleichbaren UI-Aufbaus: ein expliziter Variationspunkt, uniforme Abschnittsoperationen, weniger typabhängige Steuerung und ein klarer Composition Root.

---

## 5. Betrachtungsrahmen der Transferleistung

Die Transferleistung untersucht die **konzeptionell abgeschlossene TKX-Zielarchitektur**. Der vorliegende Quellcode dient als konkreter Strukturbeleg für UI-Komposition, Strategy-Auswahl, Ereignisbehandlung, DTOs und Presenter-Rolle. Technische Arbeitsschritte der Projektmigration – etwa die zeitliche Reihenfolge einzelner Anbindungen – sind nicht Gegenstand der Leitfrage.

Für die Argumentation wird folgende vollständige Zusammenarbeit zugrunde gelegt, ohne sie fälschlich als bereits vollständig im Repository implementiert auszugeben:

1. TKX-Komponenten erfassen und validieren Eingaben.
2. Wiederverwendbare Abschnitte synchronisieren den UI-Zustand mit typisierten DTOs.
3. Eine Nachrichtentyp-Strategy bestimmt die fachlich passende Abschnittskomposition.
4. Der Presenter koordiniert Anwendungsfälle und übersetzt zwischen View-DTOs und Anwendungsport.
5. Fachlogik und technische Verarbeitung liegen hinter Ports außerhalb der UI.

Diese Abstraktion ist methodisch zulässig, weil die Arbeit die **Eignung der Architektur und der Entwurfsmuster** untersucht, nicht den Fertigstellungsgrad eines Entwicklungsauftrags. Aussagen über Klassen und Methoden bleiben dennoch auf belegte Quellstellen beschränkt.

### 5.1 Verbleibende architektonische Trade-offs

- **Statische Registry:** sehr einfach und deterministisch, aber ein zentraler Registrierungspunkt.
- **Gemeinsames `FachdatenDTO`:** erleichtert den uniformen Abschnittsvertrag, kann bei starkem Wachstum jedoch zu einem breiten Datenmodell werden.
- **Direkte DTO-Synchronisation:** pragmatisch und lokal verständlich, verlangt aber Disziplin für beide Datenrichtungen.
- **Neuaufbau dynamischer Mehrfachabschnitte:** gewährleistet einfach einen konsistenten Zustand, kann jedoch teurer als eine inkrementelle Aktualisierung sein.
- **Konkrete Presenter-Implementierung in Konstruktoren:** hält die Verdrahtung einfach; Interface-Injektion wäre bei umfangreichen Tests und mehreren Implementierungen flexibler.

TKX liefert damit technische Voraussetzungen und begrenzt zugleich unnötige UI-Freiheiten. Strategy und Abschnittskomposition übersetzen diese Voraussetzungen in einen fachlich modularen Entwurf. Die Patterns sind hier hilfreich, weil reale Variabilität zwischen Nachrichtentypen existiert – nicht, weil TKX ihre Verwendung formal verlangt.
