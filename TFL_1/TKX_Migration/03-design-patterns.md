# Entwurfsmuster im Vergleich: TK Classic und TKX

> Autor: Copilot (2026-09-22)
> Wissensdokument 3 von 7 · Grundlage: statische Analyse der beiden Quellbäume

---

## 1. Analyseregel

Für die wissenschaftliche Einordnung wird zwischen drei Ebenen unterschieden:

1. **GoF-Muster:** Rollen und Kollaboration entsprechen dem klassischen Muster.
2. **Pattern-artige Struktur:** Ein Teil der Musteridee ist erkennbar, aber mindestens eine zentrale Rolle fehlt.
3. **Frameworkmechanismus oder Idiom:** Ähnliche Wirkung, jedoch kein eigenständiges Entwurfsmuster der Anwendung.

Diese Unterscheidung verhindert Pattern Spotting: Nicht jedes Interface ist eine Strategy, nicht jede Liste ein Composite und nicht jede `create`-Methode eine Factory Method.

---

## 2. Ergebnisübersicht

| Muster | TK Classic | TKX-Zielstruktur | Urteil |
|---|---|---|---|
| **Strategy** | nicht für den Nachrichtentyp; String-Dispatch im Agent | `TestnachrichtKonfiguration` + konkrete Konfigurationen | klar belegt und passend für Formularvarianten |
| **Composite** | echter Struct-Baum mit schmalem gemeinsamen Vertrag | flache Liste polymorpher Abschnitte + TKX-Container | TK Classic: GoF-nah; TKX: bewusst reduzierte, angemessene Komposition |
| **Observer** | VAV-Bindung und `ValueChangeListener` | Komponentenlistener als Lambdas | in beiden belegt; TKX vereinfacht Syntax, nicht das Prinzip |
| **Registry / factory-artiger Lookup** | `getStructClass()` als Framework-Fabrik-Hook | statische `TestnachrichtKonfigurationFactory` | TKX: Registry mit zentraler Instanziierung, keine GoF Factory Method |
| **Template Method** | `AbstractTestnachrichtStruct#createPanel` mit Hook | kein klares Gegenstück im Anwendungscode | im Legacy klar belegt |
| **Command/Callback** | Framework-CallValues und Listener | `Consumer<String>` und `Runnable` | im TKX als leichtgewichtiges Callback-Idiom |
| **MVP** | nicht zutreffend; ViewAgent ist MVC-ähnlich | Presenter-orientierte Zielstruktur | sinnvolle Trennung von UI-Koordination und Anwendungsfällen |

### 2.1 Bewertungskriterien

Ein Pattern ist nicht allein deshalb sinnvoll, weil seine Rollen im Code identifiziert werden können. Für die Bewertung werden fünf Fragen verwendet:

1. **Reale Variabilität:** Existieren tatsächlich mehrere austauschbare Varianten?
2. **Lokalisierung von Änderungen:** Bündelt das Pattern Änderungen an einer fachlich passenden Stelle?
3. **Reduktion von Kopplung:** Muss der Client weniger konkrete Typen, Frameworkklassen oder technische Details kennen?
4. **Verständlichkeit:** Ist die Zusammenarbeit leichter nachvollziehbar als eine direkte Lösung?
5. **Verhältnismäßigkeit:** Rechtfertigt der Nutzen zusätzliche Interfaces, Klassen und Indirektion?

Diese Kriterien passen zum untersuchten Fall, weil elf Nachrichtentypen gemeinsame und abweichende Formularabschnitte kombinieren. Variabilität und Wiederverwendung sind somit keine hypothetischen Anforderungen, sondern im Code unmittelbar sichtbar.

---

## 3. Strategy: Expliziter Variationspunkt für Nachrichtentypen

### 3.1 Problem im Legacy-Code

Der variable Aspekt „Nachrichtentyp“ ist ein String. `DatenaustauschRehaAgent` wertet ihn in fünf Methoden aus:

- `getStructClass()` – Struct-Typ auswählen,
- `fromXML()` – Business Object erzeugen,
- `doVorbelegung()` – Version bestimmen,
- `callInit()` – Daten in das Struct übertragen,
- `callUpdate()` – Struct-Daten zurückübertragen.

Der Algorithmus variiert damit nicht über austauschbare Objekte, sondern über zentrale Fallunterscheidungen. Das erschwert Erweiterungen und führt zu Shotgun Surgery.

### 3.2 Rollen in der TKX-Variante

| Strategy-Rolle | Implementierung |
|---|---|
| Strategy | `TestnachrichtKonfiguration` |
| Concrete Strategies | `AufnahmeKonfiguration`, `EntlassungKonfiguration`, `RechnungKonfiguration`, … |
| Context | `TestnachrichtApplicationUI#onNachrichtentypChanged` |
| Strategy-Auswahl | `TestnachrichtKonfigurationFactory#getKonfiguration` |
| Client-Auslöser | Combobox in `StartoperationenPanel` |

Der Vertrag kapselt Kennung, sichtbare Bezeichnung und vor allem die Zusammenstellung der Abschnitte:

```java
public interface TestnachrichtKonfiguration {
    String getNachrichtentypKennung();
    String getNachrichtentypBezeichnung();
    List<TestnachrichtAbschnitt> getAbschnitte();
}
```

Eine konkrete Strategie beschreibt den Nachrichtentyp durch Komposition:

```java
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

Der Kontext kennt keine konkrete Strategie:

```java
TestnachrichtKonfiguration konfiguration =
    TestnachrichtKonfigurationFactory.getKonfiguration(label);
aktiveAbschnitte = konfiguration.getAbschnitte();
for (TestnachrichtAbschnitt abschnitt : aktiveAbschnitte) {
    fachdatenTab.add(abschnitt.erstellePanel(fachdaten));
}
```

### 3.3 Wirkung und Grenze

**Nutzen:**

- Nachrichtentypabhängige UI-Zusammenstellung ist lokal in einer Klasse sichtbar.
- Orchestrierung arbeitet ohne Nachrichtentyp-Casts.
- Abschnitte können zwischen Strategien wiederverwendet werden.
- Die Reihenfolge der Abschnitte ist deklarativ lesbar.
- Der Wechsel der Strategie ist zur Laufzeit möglich.

**Trade-offs:**

- Ein neuer Typ muss zentral in der statischen Registry registriert werden.
- Das gemeinsame `FachdatenDTO` kann weiterhin typabhängige Erweiterungen aufnehmen.
- Das unbenutzte `TestnachrichtStrategy`-Interface ist nicht Teil der aktiven Lösung und sollte in der Arbeit nur als Entwurfsrelikt erwähnt werden.

**Bewertung:** Strategy ist hier sehr sinnvoll. Es gibt viele konkrete Varianten, sie werden zur Laufzeit ausgewählt und unterscheiden sich vor allem in der Kombination wiederverwendbarer Bausteine. Eine zentrale Fallunterscheidung wäre bei zwei stabilen Varianten einfacher, skaliert bei elf Nachrichtentypen aber schlechter. Der zusätzliche Typ pro Nachricht ist deshalb angemessene Struktur und keine unnötige Abstraktion. Das Open-Closed-Principle wird für die Orchestrierung deutlich besser unterstützt, auch wenn die zentrale Registrierung als bewusster Änderungspunkt bestehen bleibt.

---

## 4. Composite: Baum im Legacy, Komposition in TKX

### 4.1 Legacy als strukturelles Composite

`AbstractElementStruct` definiert eine Kindoperation und eine rekursive Traversierung. `MehrfachElementStruct` liefert enthaltene `EinfachElementStruct`-Objekte als Kinder. Root-Structs setzen daraus einen Baum zusammen.

| Composite-Rolle | Legacy-Typ |
|---|---|
| Component | `AbstractElementStruct` |
| Leaf | `EinfachElementStruct` und konkrete einfache Structs |
| Composite | `MehrfachElementStruct`, Root-Structs |
| Client | `DatenaustauschRehaAgent#initFromVersicherter`, `#initFromKur` |

Das ist GoF-nah, aber der gemeinsame Vertrag ist schwach: Er vereinheitlicht nur die Navigation über Kinder. Die relevanten Operationen `init`, `update` und `createPanel` besitzen unterschiedliche Signaturen und können nicht einheitlich auf dem Baum ausgeführt werden. Der Agent kompensiert dies durch `instanceof`, Casts und externe Traversierung.

### 4.2 TKX als polymorphe Abschnittskomposition

`TestnachrichtAbschnitt` vereinheitlicht genau die Operationen, welche die Orchestrierung benötigt:

```java
public interface TestnachrichtAbschnitt {
    ExpandablePanel erstellePanel(FachdatenDTO fachdaten);
    void belegeFelder(FachdatenDTO fachdaten);
}
```

`TestnachrichtApplicationUI` behandelt sämtliche Abschnitte uniform. Das ist eine bessere **operationale Uniformität** als im Legacy-Composite.

Trotzdem liegt auf Anwendungsebene kein vollständiges GoF-Composite vor:

- Das Interface besitzt keine Kinderoperation.
- Kein `TestnachrichtAbschnitt` verwaltet Kinder desselben Component-Typs.
- Mehrfachabschnitte verwalten DTO-Listen und TKX-Komponenten, nicht `TestnachrichtAbschnitt`-Kinder.
- Der Root ist eine `List<TestnachrichtAbschnitt>`, kein Component-Objekt.

Treffende Formulierung für die TFL:

> Die Migration ersetzt einen echten, aber operational schwachen Struct-Baum durch eine flache polymorphe Abschnittskomposition. Zusammen mit dem hierarchischen TKX-Containermodell entsteht Composite-artiges Verhalten, jedoch kein vollständiges anwendungseigenes GoF-Composite.

### 4.3 Wissenschaftlich interessante Umkehrung

Der Vergleich zeigt, dass die bloße Existenz eines Baumtyps weniger wichtig sein kann als ein sinnvoller gemeinsamer Operationsvertrag:

- Legacy: **starke Baumstruktur, schwache Verhaltensuniformität**.
- TKX: **flache Anwendungsstruktur, starke Verhaltensuniformität**.

Dies relativiert die These „Composite wurde erst durch TKX eingeführt“. Richtig ist: TKX erleichtert die **Komposition wiederverwendbarer UI-Bausteine**; das klassische Composite war strukturell bereits im Legacy vorhanden.

**Bewertung:** Die flache Abschnittskomposition ist für diesen Anwendungsfall sinnvoller als ein vollständiges GoF-Composite. Die Anwendung benötigt eine geordnete Folge von Formularabschnitten, aber keine beliebig tiefen Abschnittsbäume mit rekursiven Operationen. Das reduzierte Interface enthält genau die benötigten Operationen. Patternwissen hilft hier auch dabei, ein Muster bewusst **nicht vollständig** umzusetzen, wenn eine kleinere Form denselben fachlichen Nutzen mit weniger Komplexität erreicht.

---

## 5. Observer und Datenfluss

### 5.1 Legacy

Legacy-Widgets binden sich über `setHeldValue(...)` an `ViewAgentValue`-Objekte. `ValueChangeListener` und `CallValueChangeListener` reagieren auf Änderungen. Beispielrollen:

| Observer-Rolle | Legacy |
|---|---|
| Subject | `ViewAgentValue`, `CallVAV`, `CallValue` |
| Observer | `ValueChangeListener`, `CallValueChangeListener` |
| Concrete Observer | anonyme innere Listener in Panel- und Struct-Klassen |

Die Bindung ist frameworknah und teilweise bidirektional, koppelt das Presentation Model aber eng an das ViewAgent-Framework.

### 5.2 TKX

TKX-Komponenten publizieren Ereignisse, auf die Lambda-Listener reagieren:

```java
aufnahmedatum.addValueChangeListener(
    event -> fachdaten.setAufnahmedatum(event.getSource().getValue())
);
```

Die Gegenrichtung erfolgt nicht automatisch über ein beobachtbares DTO, sondern manuell:

```java
if (aufnahmedatum != null && fachdaten.getAufnahmedatum() != null) {
    aufnahmedatum.setValue(fachdaten.getAufnahmedatum());
}
```

Daraus folgt:

- TKX reduziert Listener-Boilerplate durch Lambdas.
- Das Observer-Prinzip bleibt gleich.
- Es gibt im untersuchten Code **keine vollständige bidirektionale Datenbindung**.
- Manuelle Synchronisation kann Inkonsistenzen und Wiederholungen erzeugen.

**Bewertung:** Observer ist für interaktive Oberflächen grundsätzlich angemessen, weil Eingaben ereignisgetrieben verarbeitet werden. TKX liefert diesen Mechanismus als Frameworkkonvention; eine zusätzliche anwendungseigene Observer-Hierarchie wäre unnötig. Sinnvoll ist die direkte Nutzung der TKX-Listener. Der Gestaltungsspielraum liegt darin, Listener auf lokale Zustandsänderungen zu begrenzen und übergreifende Aktionen an Presenter oder Callbacks zu delegieren.

---

## 6. Registry und Erzeugungsmuster

`TestnachrichtKonfigurationFactory` instanziiert Konfigurationen zentral in einem statischen Block, registriert sie in einer `LinkedHashMap`, liefert sie per Label und stellt die verfügbaren Labels bereit. Fachlich ist dies vor allem:

- eine **Registry** als zentraler Katalog bekannter Konfigurationen,
- verbunden mit zentraler Instanziierung und einem **factory-artigen Lookup**,
- eingeschränkt ein **Service Locator**, weil der Client den Katalog aktiv nach einem Objekt fragt.

Es ist **keine GoF Factory Method**, weil keine abstrakte Creator-Klasse eine überschreibbare Erzeugungsmethode definiert. Auch eine klassische Simple Factory, die bei jedem Aufruf ein neues Objekt erzeugt, liegt nicht vor. Die im Projekt verwendete Kurzbezeichnung „Registry/Simple Factory“ beschreibt die Wirkung, technisch genauer ist „statische Registry mit zentraler Instanziierung und factory-artigem Lookup“.

Vorteile:

- zentrale, geordnete Liste der angebotenen Nachrichtentypen,
- einheitliche Fehlerbehandlung für unbekannte Labels,
- keine Typkaskade im Client.

Nachteile:

- statischer globaler Zustand,
- Erweiterung erfordert Änderung des `static`-Blocks,
- erschwerte Isolation und Austauschbarkeit in Unit-Tests,
- Label dient als Schlüssel und koppelt Anzeige an technische Auswahl.

Mögliche Weiterentwicklung: Instanzen am Composition Root injizieren und nach einer stabilen Kennung indizieren; Labels nur für die Darstellung verwenden.

**Bewertung:** Die Registry ist angesichts einer kleinen, zur Entwicklungszeit bekannten Menge von Nachrichtentypen eine pragmatische Lösung. Eine dynamische Plugin-Infrastruktur oder reflexionsbasierte Erkennung wäre komplexer und bietet für den belegten Anwendungsfall keinen erkennbaren Zusatznutzen. Der zentrale Katalog ist daher nicht nur ein OCP-Kompromiss, sondern auch eine leicht verständliche Übersicht aller Varianten.

---

## 7. Template Method im Legacy-Code

`AbstractTestnachrichtStruct#createPanel()` gibt das Algorithmusskelett vor:

1. Administration-Panel erstellen,
2. Kopf-, Admin- und Dokumentabschnitte hinzufügen,
3. Fachdaten-Tab erstellen,
4. Administration-Tab erstellen.

Der variable Schritt ist die abstrakte Hook-Methode:

```java
protected abstract TkPanel createFachdatenPanel(TkPanel main);
```

`AufnahmeTestnachrichtStruct`, `EntlassungTestnachrichtStruct` usw. implementieren diesen Schritt. Das entspricht dem Template Method Pattern. Auch `init()` und `update()` werden durch `super.init(...)` beziehungsweise `super.update(...)` erweitert.

Die TKX-Lösung ersetzt dieses Vererbungsmodell im Formularaufbau durch **Objektkomposition**: Eine Strategy liefert Abschnitte, die der Kontext iteriert. Das entspricht dem Designgrundsatz „favor object composition over class inheritance“, ohne dass Vererbung generell falsch wäre.

**Bewertung:** Template Method ist im homogenen Struct-Vererbungsbaum von TK Classic sinnvoll. In TKX passt dieselbe tiefe Vererbung weniger gut, weil `UIFactory` Komponenten als Interfaces liefert und Nachrichtentypen Abschnitte flexibel kombinieren. Die Ablösung ist keine generelle Überlegenheit von Komposition, sondern eine bessere Passung zum neuen Framework und zum Variationsbedarf der Anwendung.

---

## 8. Callback-/Command-Idiom

`StartoperationenPanel` erhält zwei Funktionsobjekte:

```java
Consumer<String> onNachrichtentypGewaehlt;
Runnable onFelderBelegen;
```

Dadurch muss das Panel weder Factory noch Abschnittstypen kennen. Dies ähnelt einem Command Pattern, ist aber treffender als **Callback-/Command-Idiom** zu bezeichnen: Es existieren keine benannten Command-Objekte mit eigener Lebensdauer, Historie oder Undo-Funktion.

---

## 9. Verhältnis der Muster zu TKX

TKX **ermöglicht und begünstigt** die Lösung durch:

- uniforme Komponenteninterfaces,
- Container mit `add` und `removeAll`,
- `UIFactory` statt konkreter Widgetklassen,
- Lambda-kompatible Ereignisschnittstellen,
- standardisierte Layout- und Validierungsprimitive,
- einen klaren Einstieg über `ApplicationUIConstructor`.

TKX **erzwingt** jedoch weder Strategy, Abschnittskomposition noch MVP. Auch in TKX wäre eine monolithische Klasse mit `if`-Kaskaden möglich. Der Qualitätsgewinn entsteht aus dem Zusammenspiel von Frameworkmöglichkeiten und bewussten Entwurfsentscheidungen.

### 9.1 Wie stark gibt TKX die Muster vor?

| TKX-Eigenschaft | Naheliegendes Muster/Prinzip | Grad der Vorgabe |
|---|---|---|
| Komponenten werden über `UIFactory` als Interfaces erzeugt | Objektkomposition, Dependency Inversion | stark begünstigt, aber nicht erzwungen |
| Container akzeptieren uniforme UI-Komponenten | Abschnittskomposition/Composite-artige Struktur | stark begünstigt |
| Value-Change- und Aktivierungslistener | Observer | praktisch vorgegebenes Interaktionsmodell |
| `ApplicationUIConstructor` | Composition Root, MVP-Verdrahtung | klarer struktureller Anker |
| standardisierte Anwendungstypen und Layouts | Framework Inversion, Konsistenz | durch Konvention vorgegeben |
| `ComponentId` und abstrakte UI-Elemente | Testbarkeit und stabile Identifikation | technisch unterstützt |
| keine Ableitung konkreter TKX-Komponenten | Komposition statt Widget-Vererbung | bewusst durch API-Design gelenkt |

TKX wirkt damit auf zwei Ebenen. **Ermöglichend** stellt es uniforme Schnittstellen und Ereignisse bereit. **Lenkend** begrenzt es konkrete Widget-Vererbung und individuelles Layout. Die anwendungsspezifischen Patterns bleiben dennoch Entwurfsentscheidungen: Erst `TestnachrichtKonfiguration` macht den Nachrichtentyp zur Strategy, und erst `TestnachrichtAbschnitt` schafft einen fachlich benannten Komponentenvertrag.

### 9.2 Gesamtnutzen und mögliche Übermodellierung

| Lösung | Nutzen im konkreten Fall | Risiko unnötiger Komplexität | Gesamturteil |
|---|---|---|---|
| Strategy je Nachrichtentyp | sehr hoch bei elf kombinierbaren Varianten | zusätzliche kleine Klassen | klar angemessen |
| flache Abschnittskomposition | sehr hoch durch Wiederverwendung und uniforme Orchestrierung | gering | klar angemessen |
| vollständiges GoF-Composite | kein belegter Bedarf an rekursiven Abschnittsbäumen | zusätzliche Kinder- und Traversierungslogik | bewusst nicht erforderlich |
| TKX-Observer/Listener | notwendig für interaktive Eingaben | Logik kann sich in Listenern verteilen | angemessen bei lokaler Listenerlogik |
| statische Registry mit factory-artigem Lookup | einfacher, deterministischer Zugriff | zentraler Registrierungspunkt | pragmatisch angemessen |
| MVP | trennt UI-Koordination und Anwendungsfälle | Mapping- und Schnittstellenaufwand | für fachlich relevante Aktionen sinnvoll |
| eigene Widget-Vererbung | kaum Nutzen bei `UIFactory`-Interfaces | starke Frameworkkopplung | in TKX bewusst zu vermeiden |

---

## 10. Kernaussagen für die wissenschaftliche Arbeit

1. Der Legacy-Code enthält bereits mehrere Muster; Migration ist keine Bewegung von „musterlos“ zu „musterbasiert“.
2. Der größte belegte Fortschritt ist die explizite Strategy für die Nachrichtentyp-abhängige UI-Zusammenstellung.
3. Das klassische Composite ist im Legacy strukturell stärker ausgeprägt; TKX verbessert dagegen den gemeinsamen Operationsvertrag.
4. Observer existiert in beiden Varianten. TKX modernisiert hauptsächlich API und Syntax.
5. Die Factory ist präzise als statische Registry/Simple Factory zu bezeichnen.
6. TKX schafft günstige Rahmenbedingungen, die Muster selbst bleiben Entscheidungen der Anwendungsarchitektur.
7. Die eingesetzten Patterns sind überwiegend angemessen, weil sie reale Varianten und Wiederverwendung strukturieren; ein vollständiges Composite wäre dagegen übermodelliert.
8. Für die Transferleistung wird die konzeptionell abgeschlossene Zielarchitektur betrachtet, während konkrete Codebelege weiterhin exakt als solche gekennzeichnet werden.
