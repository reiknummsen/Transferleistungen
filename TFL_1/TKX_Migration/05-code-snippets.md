# Kuratierte Codebelege für NotebookLM

> Autor: Copilot (2026-09-22)
> Wissensdokument 5 von 7 · Ausschnitte wurden für Lesbarkeit gekürzt; `/* … */` markiert Auslassungen

---

## Nutzungshinweis

Die folgenden Ausschnitte ersetzen nicht den vollständigen Quellcode. Sie sind ausgewählt, um die zentralen Thesen der Transferleistung zu belegen. Dateipfade und Methoden dienen als Fundstellen. Zeilennummern beziehen sich auf den untersuchten Stand und können sich später verschieben.

---

## Beleg 1: Stringbasierter Einstieg im Legacy-System

**Datei:** `kur.ui.qs/.../testnachricht/DatenaustauschRehaStep.java`, `init()` und `getAgent()`

```java
nachrichtentyp = (String) getParameter("Nachrichtentyp").getContent();
/* Laden von Kur und Versicherter */

if (agent == null) {
    agent = new DatenaustauschRehaAgent(kur, versicherter, nachrichtentyp);
}
return agent;
```

**Belegt:** Der Nachrichtentyp wird als String von der Anwendungskapsel bis in den Agent weitergereicht.

**Nicht belegt:** Dass Strings grundsätzlich ungeeignet wären; problematisch wird die Kombination mit mehrfach wiederholtem Dispatch.

---

## Beleg 2: Erste von fünf Legacy-Typkaskaden

**Datei:** `kur.ui.qs/.../testnachricht/DatenaustauschRehaAgent.java`, `getStructClass()`

```java
@Override
public Class<?> getStructClass() {
    if (isAufnahme()) {
        return AufnahmeTestnachrichtStruct.class;
    }
    if (isAbsageEinrichtung()) {
        return AbsageEinrichtungTestnachrichtStruct.class;
    }
    /* neun weitere Typen */
    throw new AgentCreateException(nachrichtentyp + " ist unbekannter Nachrichtentyp");
}
```

**Belegt:** Zentraler String-Dispatch und Framework-Fabrik-Hook.

**Ergänzende Fundstellen:** `fromXML()`, `doVorbelegung()`, `callInit()`, `callUpdate()` wiederholen denselben Variationspunkt.

---

## Beleg 3: Doppelter Downcast im Legacy-Agent

**Datei:** `DatenaustauschRehaAgent.java`, `callInit()` und `callUpdate()`

```java
if (isAufnahme()) {
    ((AufnahmeTestnachrichtStruct) struct)
        .init((KurAufnahmeImpl) kurNachricht);
    return;
}
```

```java
if (isAufnahme()) {
    ((AufnahmeTestnachrichtStruct) struct)
        .update((KurAufnahmeImpl) kurNachricht);
    return;
}
```

**Belegt:** Struct- und BO-Typ werden parallel anhand desselben Strings angenommen. Der Compiler kann die korrekte Paarung nicht über den allgemeinen Feldtyp garantieren.

---

## Beleg 4: Legacy-Composite und externe Rekursion

**Datei:** `AbstractElementStruct.java`

```java
public abstract class AbstractElementStruct extends AbstractVAVStruct {

    public abstract AbstractElementStruct[] getChildrenStructs();

    public static void getChildrenStructRekursiv(
        List<AbstractElementStruct> result,
        AbstractElementStruct root
    ) {
        for (AbstractElementStruct struct : root.getChildrenStructs()) {
            result.add(struct);
            getChildrenStructRekursiv(result, struct);
        }
    }
}
```

**Belegt:** Ein hierarchischer Component-Vertrag und rekursive Traversierung sind bereits im Legacy-Code vorhanden.

**Kritik:** Der Vertrag vereinheitlicht Navigation, nicht die fachlich relevanten Operationen `init`, `update` und `createPanel`.

---

## Beleg 5: Template Method im Legacy-System

**Datei:** `AbstractTestnachrichtStruct.java`, `createPanel()`

```java
@Override
public void createPanel(TkPanel main, JTabbedPane tabbedPane) {
    TkPanel administrationsdatenPanel = new TkPanel();
    /* gemeinsame Kopf-, Admin- und Dokument-Panels */

    tabbedPane.addTab(
        "[1] Fachdaten",
        new TkEditScrollPane(createFachdatenPanel(main))
    );
    tabbedPane.addTab(
        "[2] Administration",
        new TkEditScrollPane(administrationsdatenPanel)
    );
}

protected abstract TkPanel createFachdatenPanel(TkPanel main);
```

**Belegt:** Die Basisklasse legt das Algorithmusskelett fest; Unterklassen implementieren den variablen Schritt.

---

## Beleg 6: Vermischung von Daten, Mapping und Swing-Layout

**Datei:** `aufnahme/AufnahmeTestnachrichtStruct.java`

```java
public class AufnahmeTestnachrichtStruct
    extends AbstractTestnachrichtStruct<KurAufnahmeImpl> {

    private final ViewAgentValue<Datum> aufnahmedatum;

    @Override
    public void init(KurAufnahmeImpl aufnahme) {
        super.init(aufnahme);
        aufnahmedatum.set(aufnahme.getAufnahmedatum());
        /* weiteres BO -> Struct-Mapping */
    }

    @Override
    public void update(KurAufnahmeImpl nachricht) {
        super.update(nachricht);
        nachricht.setAufnahmedatum(aufnahmedatum.get());
        /* weiteres Struct -> BO-Mapping */
    }

    @Override
    protected TkPanel createFachdatenPanel(TkPanel main) {
        /* Swing-Panel- und Layout-Erzeugung */
    }
}
```

**Belegt:** Ein Struct vereint Presentation State, bidirektionales BO-Mapping und UI-Erzeugung.

---

## Beleg 7: TKX-Einstieg als Composition Root

**Datei:** `testdaten-kur.ui/.../testandwendung/TestnachrichtUIConstructor.java`

```java
@Override
public ApplicationUI create(Map<String, Object> parameter) {
    TestnachrichtPresenterImpl presenter =
        new TestnachrichtPresenterImpl(parameter);

    return new TestnachrichtApplicationUI(presenter).getApp();
}
```

**Belegt:** Der Einstieg und die Objektverdrahtung sind lokal sichtbar.

**Bewertung:** Der definierte Einstieg eignet sich als Composition Root. Die direkte Verwendung der Implementierung hält die Verdrahtung einfach; eine Injektion über das Interface wäre bei mehreren Presenter-Implementierungen oder umfangreicher Isolation flexibler.

---

## Beleg 8: Aktive Strategy-Schnittstelle

**Datei:** `konfiguration/TestnachrichtKonfiguration.java`

```java
public interface TestnachrichtKonfiguration {
    String getNachrichtentypKennung();
    String getNachrichtentypBezeichnung();
    List<TestnachrichtAbschnitt> getAbschnitte();
}
```

**Belegt:** Nachrichtentypabhängige Formularzusammensetzung besitzt einen expliziten polymorphen Vertrag.

---

## Beleg 9: Konkrete Strategy als Komposition

**Datei:** `konfiguration/AufnahmeKonfiguration.java`, `getAbschnitte()`

```java
@Override
public List<TestnachrichtAbschnitt> getAbschnitte() {
    return List.of(
        new RehabilitandAbschnitt(),
        new LebendspenderAbschnitt(),
        new KommunikationsdatenAbschnitt(),
        new AufnahmeAbschnitt(),
        new AufnahmediagnosenAbschnitt(),
        new KindMutterKindMassnahmenMehrfachAbschnitt(/* ... */),
        new BegleitpersonenMehrfachAbschnitt(/* ... */)
    );
}
```

**Belegt:** Struktur und Reihenfolge eines Nachrichtentyps sind an einer Stelle lesbar; Abschnitte werden wiederverwendet.

---

## Beleg 10: Registry statt Typkaskade

**Datei:** `konfiguration/TestnachrichtKonfigurationFactory.java`

```java
private static final Map<String, TestnachrichtKonfiguration>
    KONFIGURATIONEN = new LinkedHashMap<>();

static {
    registriere(new AufnahmeKonfiguration());
    registriere(new AbsageEinrichtungKonfiguration());
    /* weitere Konfigurationen */
}

public static TestnachrichtKonfiguration getKonfiguration(String typ) {
    TestnachrichtKonfiguration konfiguration = KONFIGURATIONEN.get(typ);
    if (konfiguration == null) {
        throw new IllegalArgumentException("Unbekannter Nachrichtentyp: " + typ);
    }
    return konfiguration;
}
```

**Belegt:** Lookup und verfügbare Strategien sind zentralisiert.

**Einordnung:** Statische Registry mit zentraler Instanziierung und factory-artigem Lookup; keine GoF Factory Method.

**Bewertung:** Neue Typen werden bewusst an einer zentralen Stelle registriert. Das ist ein kleiner OCP-Kompromiss, bietet aber einen einfachen und vollständigen Überblick über alle unterstützten Varianten.

---

## Beleg 11: Uniformer Abschnittsvertrag

**Datei:** `abschnitt/TestnachrichtAbschnitt.java`

```java
public interface TestnachrichtAbschnitt {
    ExpandablePanel erstellePanel(FachdatenDTO fachdaten);
    void belegeFelder(FachdatenDTO fachdaten);
}
```

**Belegt:** Alle Fachabschnitte unterstützen dieselben beiden Operationen.

**Aussagegrenze:** Das Interface enthält keine Kinderoperation. Es ist daher allein noch kein vollständiges GoF-Composite.

---

## Beleg 12: Strategy-Auswahl und polymorphe Komposition

**Datei:** `TestnachrichtApplicationUI.java`, `onNachrichtentypChanged()`

```java
private void onNachrichtentypChanged(String label, Tab fachdatenTab) {
    TestnachrichtKonfiguration konfiguration =
        TestnachrichtKonfigurationFactory.getKonfiguration(label);
    aktiveAbschnitte = konfiguration.getAbschnitte();
    fachdatenTab.removeAll();

    FachdatenDTO fachdaten = presenter.getDTO().getFachdatenDTO();
    for (TestnachrichtAbschnitt abschnitt : aktiveAbschnitte) {
        fachdatenTab.add(abschnitt.erstellePanel(fachdaten));
    }
}
```

**Belegt:** Der Kontext wählt die Strategy und behandelt alle Abschnitte ohne konkreten Typ und ohne Cast.

**Bewertung:** Der UI-bezogene Variationspunkt ist für den Formularaufbau aus der zentralen Orchestrierung ausgelagert. Dieselbe Entwurfsregel kann für weitere typabhängige Verarbeitung hinter geeigneten Ports und Strategien verwendet werden.

---

## Beleg 13: TKX-Observer und manuelle Gegenrichtung

**Datei:** `abschnitt/AufnahmeAbschnitt.java`

```java
aufnahmedatum = UIFactory.newDateInput(
    ComponentId.named("Aufnahmedatum")
);
aufnahmedatum.setName("Aufnahmedatum");
aufnahmedatum.addValueChangeListener(
    event -> fachdaten.setAufnahmedatum(event.getSource().getValue())
);
```

```java
@Override
public void belegeFelder(FachdatenDTO fachdaten) {
    if (aufnahmedatum != null && fachdaten.getAufnahmedatum() != null) {
        aufnahmedatum.setValue(fachdaten.getAufnahmedatum());
    }
}
```

**Belegt:** Widgetänderungen werden ereignisbasiert ins DTO geschrieben; die Rückrichtung wird explizit programmiert.

**Folgerung:** Observer ja, vollständiges automatisches Two-Way-Binding nein.

---

## Beleg 14: Callback-Entkopplung im Startpanel

**Datei:** `StartoperationenPanel.java`

```java
public StartoperationenPanel(
    TestnachrichtPresenterImpl presenter,
    Collection<String> nachrichtentypen,
    Consumer<String> onNachrichtentypGewaehlt,
    Runnable onFelderBelegen
) {
    /* Zuweisungen */
}
```

```java
combobox.addValueChangeListener(event -> {
    String gewaehlterTyp = event.getSource().getValue();
    if (gewaehlterTyp != null) {
        onNachrichtentypGewaehlt.accept(gewaehlterTyp);
    }
});
```

**Belegt:** Das Panel kennt die Reaktion auf einen Typwechsel nicht; sie wird als Verhalten übergeben.

**Einordnung:** Leichtgewichtiges Callback-/Command-Idiom.

---

## Beleg 15: Mehrfachabschnitt und vollständiger Neuaufbau

**Datei:** `abschnitt/BegleitpersonenMehrfachAbschnitt.java`

```java
private void aktualisierePanel() {
    panel.removeAll();
    for (int index = 0; index < begleitpersonen.size(); index++) {
        panel.add(erstelleBegleitpersonPanel(
            begleitpersonen.get(index), index
        ));
    }
    /* Hinzufügen-/Entfernen-Aktionen neu anlegen */
}
```

**Belegt:** Wiederholbare Daten werden über einen wiederverwendbaren Baustein dargestellt; bei Änderungen wird die Teil-UI vollständig rekonstruiert.

**Aussagegrenze:** Ohne Laufzeitmessung ist dies nur ein Performance- und Zustandsrisiko, kein nachgewiesener Defekt.

---

## Beleg 16: Presenter als Koordinationspunkt

**Dateien:** `TestnachrichtPresenterImpl.java` und `StartoperationenPanel.java`

```java
public class TestnachrichtPresenterImpl implements TestnachrichtPresenter {
    private TestnachrichtDTO dto;

    public void ermitteleDatenZurFallnummer(String fallnummer) {
        /* Koordination der Vorbelegung */
    }

    public TestnachrichtDTO getDTO() {
        return dto;
    }
}
```

```java
presenter.ermitteleDatenZurFallnummer(fallnummer.getValue());
belegeFelder(presenter.getDTO());
onFelderBelegen.run();
```

**Belegt:** Das Panel delegiert eine übergreifende Aktion an den Presenter und übernimmt anschließend die gelieferten View-Daten.

**Architekturfolgerung:** Im MVP-Zielbild bildet der Presenter die Grenze zwischen TKX-Interaktion und Anwendungsfällen. Elementare Feldänderungen dürfen weiterhin lokal im Abschnitt bleiben; andernfalls würde der Presenter unnötig aufgebläht.

---

## Beleg 17: TKX-Containerkomposition statt Swing-Layoutgerüst

**Datei:** `DatenPanel.java`

```java
Formular formular = UIFactory.newFormular(ComponentId.of("DatenFormular"));
TabPanel tabPanel = UIFactory.newTabPanel(ComponentId.of("DatenTabPanel"));
tabPanel.add(fachdatenTab);
tabPanel.add(administrationPanel.getTab());
formular.add(tabPanel);

panel = UIFactory.newExpandablePanel(ComponentId.of("DatenPanel"));
panel.setTitle("Daten").setExpandable(false);
panel.add(formular);
```

**Belegt:** Die Oberfläche wird aus abstrakten, uniformen TKX-Containern zusammengesetzt. Anwendungscode benötigt weder konkrete Swing-Komponenten noch Guide-Line-Objekte.

**Patternbezug:** Das Framework liefert die technische Containerhierarchie; die Anwendung ergänzt mit `TestnachrichtAbschnitt` den fachlich benannten Komponentenvertrag.

---

## Beleg 18: TKX-Komponenten und feldnahe Validierung

**Datei:** `StartoperationenPanel.java`, `createFallnummerFormular()`

```java
fallnummer = UIFactory.newTextInput(ComponentId.of("Fallnummer"))
    .setName("Fallnummer")
    .setMandatory(true)
    .addValidator(v -> {
        if (v.getValue().length() != 13) {
            return ValidationResult.invalid(
                "Die Fallnummer muss 13 Zeichen lang sein."
            );
        }
        return ValidationResult.valid();
    });
```

**Belegt:** TKX unterstützt lokale, explizite Validierung und Fluent-Konfiguration.

**Aussagegrenze:** Einzelne Regeln beweisen keine vollständige Validierungsparität.

---

## Empfohlene Zitatpaare

Für eine kompakte Vorher-Nachher-Darstellung eignen sich:

1. **Variationspunkt:** Beleg 2/3 gegen Beleg 8/10/12.
2. **Composite-Diskussion:** Beleg 4 gegen Beleg 11/12.
3. **Observer:** Legacy-`setHeldValue` aus Dokument 01 gegen Beleg 13.
4. **Architektur:** Beleg 6 gegen Beleg 7/16.
5. **Framework-Fit:** Legacy-Layout aus Beleg 6 gegen TKX-Composition-Root und Container aus Beleg 7/17.
