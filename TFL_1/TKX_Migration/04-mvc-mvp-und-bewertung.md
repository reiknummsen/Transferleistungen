# Vom ViewAgent-Modell zum MVP-Zielbild: Architektur und Bewertung

> Autor: Copilot (2026-09-22)
> Wissensdokument 4 von 7 · Bewertungsgrundlage: statische Codeanalyse, kein Laufzeit- oder Nutzertest

---

## 1. Betrachtungsrahmen

Die Bezeichnungen MVC und MVP dürfen nicht allein aus Klassennamen abgeleitet werden. Entscheidend sind Verantwortungen und Abhängigkeitsrichtungen. Die Transferleistung betrachtet deshalb die durch den Code erkennbare Struktur als Grundlage einer **konzeptionell abgeschlossenen MVP-orientierten Zielarchitektur**.

Für die Arbeit sollte konsequent unterschieden werden:

- **Legacy-Ist:** TK-eigenes ViewAgent-Modell, MVC-ähnlich, aber mit vermischten Verantwortlichkeiten.
- **TKX-Strukturbeleg:** DTO/Presentation Model, Callbacks, modulare Views und Presenter-Typen.
- **TKX-Zielbetrachtung:** über den Presenter koordinierte Anwendungsfälle mit Ports zur Domäne.

---

## 2. TK Classic: MVC-ähnliches ViewAgent-Modell

### 2.1 Rollenverteilung

```text
.ac / .wi
   |
   v
DatenaustauschRehaStep
   | lädt Kur / Versicherter und erzeugt
   v
DatenaustauschRehaAgent <------> Business Objects, XML, EAI, Dateisystem
   |
   v
Struct-Baum <------ ViewAgentValue / Listener ------> DatenaustauschRehaPanel
                                                   ^
                                                   |
                                      DatenaustauschRehaView
```

| MVC-nahe Rolle | Konkrete Elemente | Problem |
|---|---|---|
| Controller | `DatenaustauschRehaAgent`, teilweise `DatenaustauschRehaStep` | Agent übernimmt zusätzlich Fach-, Mapping- und Infrastrukturaufgaben |
| Model/Presentation Model | Structs und `ViewAgentValue` | Structs erzeugen zusätzlich Swing-Panels |
| View | `DatenaustauschRehaView`, `DatenaustauschRehaPanel`, `createPanel`-Methoden der Structs | Darstellung ist über mehrere Ebenen verteilt |

Die Abhängigkeiten sind zyklisch beziehungsweise wechselseitig: Das Panel kennt den Root-Struct; Structs erzeugen Panels; der Agent traversiert und castet Structs. Daher ist „MVC-ähnlich“ präziser als „sauberes MVC“.

### 2.2 God Class als Folge unklarer Grenzen

`DatenaustauschRehaAgent` umfasst rund 623 Zeilen und verbindet:

- Auswahl des Nachrichtentyps,
- Erzeugung konkreter Business Objects,
- Vorbelegung,
- Struct-Mapping,
- XML-Import und -Export,
- Dateiverarbeitung,
- Import in die Verarbeitungsstrecke,
- Auswahl direkter oder EAI-basierter Folgeprüfung.

Die Klasse ist damit nicht nur Controller. Sie ist zugleich Application Service, Mapper, Factory und Infrastrukturkoordinator. Dies erschwert isolierte Tests und erhöht die Änderungswahrscheinlichkeit.

---

## 3. TKX: Daten- und Kontrollfluss der UI-Struktur

```text
TestnachrichtUIConstructor
   |
   +--> new TestnachrichtPresenterImpl(parameter)
   |
   +--> new TestnachrichtApplicationUI(presenter)
              |
              +--> StartoperationenPanel --Callbacks--> ApplicationUI
              |          |
              |          +--> presenter.ermitteleDatenZurFallnummer(...)
              |
              +--> DatenPanel
                       +--> AdministrationPanel
                       +--> Fachdaten-Tab
                                +--> List<TestnachrichtAbschnitt>
                                         |
                                         +--> Listener schreiben direkt in FachdatenDTO
```

### 3.1 Bausteine einer MVP-orientierten Struktur

- Ein Presenter-Typ und eine Implementierung bilden den Koordinationspunkt.
- UI-Daten liegen in DTOs statt direkt in persistenten Business Objects.
- Der `ApplicationUIConstructor` ist ein klarer Composition Root.
- `StartoperationenPanel` delegiert Aktionen über Callbacks.
- Der Presenter ist der geeignete Zugriffspunkt auf Anwendungsports.

### 3.2 Sinnvoller Zuschnitt des Presenters

Ein Presenter muss nicht jede elementare Feldänderung vermitteln. Für diese Anwendung ist eine Mischform sinnvoll:

- **Lokale UI-Zustände** wie der Wert eines Eingabefelds werden direkt zwischen TKX-Komponente und DTO synchronisiert.
- **Übergreifende Aktionen** wie Vorbelegung, Erzeugung, Import oder fachliche Validierung werden über den Presenter koordiniert.
- **Darstellungsentscheidungen** wie die Auswahl und Reihenfolge von Abschnitten werden durch Strategy und View-Orchestrierung getragen.
- **Fachlogik** bleibt hinter einem Anwendungsport und wird nicht in Listener verlagert.

Diese Aufteilung entspricht eher einem **Supervising Presenter mit Presentation Model** als einem vollständig passiven View. Sie vermeidet einen übergroßen Presenter und hält einfache Interaktionen lokal, ohne fachliche Anwendungsfälle an die View zu koppeln.

---

## 4. MVP- und Ports-&-Adapter-Zielbild

```text
TKX View / Abschnitte
        |
        v
TestnachrichtPresenter (Interface)
        |
        v
TestnachrichtPresenterImpl          [UI-Adapter]
        |
        v
TestnachrichtAnwendungsport         [Input-Port in kur.biz]
        |
        v
Application Service / Domäne
        |
        +--> Output-Port --> XML-/Datei-Adapter
        +--> Output-Port --> BO-/Persistenz-Adapter
        +--> Output-Port --> EAI-/Import-Adapter
```

Die Zielarchitektur bedeutet:

- Die View hängt nur am Presenter-Vertrag.
- Der Presenter mappt UI-DTOs auf Input-Port-Modelle und zurück.
- Fachliche Validierung und Nachrichtenerzeugung liegen in der Domäne beziehungsweise im Application Service.
- Datei- und EAI-Zugriffe liegen hinter Output-Ports.
- TKX-spezifische Typen verlassen den UI-Adapter nicht.
- Die Domäne kennt weder `UIFactory` noch Panels, Widgets oder `ApplicationUI`.

### 4.1 MVP-Varianten

Für diese Anwendung ist ein **Supervising Presenter / Presentation Model-Hybrid** plausibel: Einfache Feldänderungen können direkt zwischen Widget und DTO synchronisiert werden; komplexe Aktionen wie Fallvorbelegung, Nachrichtentypwechsel, XML-Erzeugung und Import laufen über den Presenter. Ein vollständig passiver View würde mehr Schnittstellen- und Mappingcode erzeugen, aber maximale Testisolation bieten.

---

## 5. Qualitätsbewertung

Die Bewertung orientiert sich an den für Wartungssoftware besonders relevanten Eigenschaften Modularität, Analysierbarkeit, Modifizierbarkeit und Testbarkeit. Sie ist qualitativ und wird durch statische Strukturmerkmale gestützt.

### 5.1 Erweiterbarkeit

**Legacy:** Ein neuer Nachrichtentyp erfordert neue Structs sowie Änderungen in der Anwendungskapsel und an sieben bestehenden Stellen, darunter fünf Dispatch-Methoden im Agent und ein neues String-Prädikat.

**TKX-Formularteil:** Eine neue Konfiguration und neue beziehungsweise wiederverwendete Abschnitte werden ergänzt; zusätzlich erfolgt eine Registrierung im statischen Registry-Block. Die Orchestrierung bleibt unverändert.

**Gewinn:** Der Variationspunkt ist explizit, lokaler und weitgehend geschlossen gegenüber neuen UI-Konfigurationen.

**Trade-off:** `FachdatenDTO` ist ein typenübergreifendes Gesamtobjekt mit rund 670 Zeilen. Es vereinfacht den gemeinsamen Abschnittsvertrag, kann bei starkem Wachstum aber selbst zum zentralen Änderungspunkt werden. Typbezogene DTOs oder Teilmodelle wären dann eine mögliche Weiterentwicklung.

### 5.2 Kopplung

**Verbessert:**

- `TestnachrichtApplicationUI` importiert keine konkreten Nachrichtentypen.
- UI-Abschnitte arbeiten gegen `FachdatenDTO`, nicht direkt gegen `KurAufnahmeImpl` usw.
- `StartoperationenPanel` kennt über Callbacks weder Factory noch Orchestrierung.
- TKX-Interfaces verhindern anwendungsspezifische Ableitungshierarchien konkreter Widgets.

**Verbleibend:**

- Orchestrierung und Panel hängen an `TestnachrichtPresenterImpl` statt am Interface.
- Die statische Registry ist global gekoppelt.
- Mehrere Abschnittsklassen und DTOs importieren weiterhin `de.tk.biz.*`-Typen.
- Das gemeinsame `FachdatenDTO` koppelt alle Nachrichtentypen an denselben Datencontainer.

### 5.3 Kohäsion und Verantwortlichkeiten

**Verbessert:** Nachrichtentypkonfiguration, statischer Seitenrahmen, Administration, Startoperationen und einzelne Fachabschnitte sind getrennte Klassen.

**Trade-off:** `AdministrationPanel` umfasst rund 327 Zeilen, `FachdatenDTO` rund 670 Zeilen. Die grobe Verantwortungsverteilung ist klarer; bei weiterem Wachstum kann eine zusätzliche Zerlegung dieser beiden Bausteine sinnvoll werden.

### 5.4 Testbarkeit

TKX ist laut interner Frameworkdokumentation (`.github/skills/tkx/02-how-tos-grundlagen/technische-grundlagen.md`) auf leicht testbare abstrakte UI-Elemente ausgelegt. Explizite `ComponentId`s bieten stabile Anker. DTOs und Abschnittsinterfaces ermöglichen prinzipiell Tests ohne persistente BOs.

Aus der Architektur ergeben sich klar abgrenzbare Testfälle:

1. eindeutige Registry-Schlüssel und vollständige Typauswahl,
2. korrekte Abschnittsfolge je Nachrichtentyp,
3. Widget -> DTO und DTO -> Widget,
4. Mindestanzahl sowie Hinzufügen/Entfernen bei Mehrfachabschnitten,
5. Validierungsparität zum Legacy-Code,
6. Mapping-, BO-/XML-Roundtrip- und Importverhalten hinter den Ports.

Die statische Registry ist in Tests weniger flexibel als eine injizierte Registry. Dafür ist sie klein, deterministisch und ohne zusätzliche Infrastruktur nutzbar. `ComponentId`, DTOs und kleine Abschnittsverträge sind dagegen direkte Testbarkeitsvorteile von TKX und der gewählten Zerlegung.

### 5.5 Lesbarkeit und Analysierbarkeit

`AufnahmeKonfiguration#getAbschnitte` zeigt die Formularstruktur auf wenigen Zeilen. Im Legacy-Code verteilt sich dieselbe Information über Konstruktor, `getChildrenStructs`, `createFachdatenPanel`, `init`, `update` und Agent-Dispatch. Die TKX-Struktur verbessert daher die lokale Nachvollziehbarkeit.

Gegenläufig wirken lange DTOs, viele ähnlich aufgebaute Abschnittsklassen und manuelle `belegeFelder`-Blöcke. Generische Hilfen können Boilerplate reduzieren, dürfen jedoch nicht durch zu viele boolesche Konfigurationsparameter unverständlich werden.

### 5.6 Laufzeit und Zustandsmanagement

Mehrfachabschnitte rufen bei Änderungen `panel.removeAll()` auf und erzeugen alle Kindkomponenten neu. Vorteile sind eine einfache und deterministische Rekonstruktion. Nachteile sind:

- potenziell unnötige Objekterzeugung,
- möglicher Verlust von UI-Zustand wie Fokus,
- steigende Kosten bei großen Listen,
- erschwerte inkrementelle Aktualisierung.

Ohne Messung darf daraus kein konkretes Performanceproblem behauptet werden. Es ist ein plausibles Risiko und Kandidat für Profiling.

### 5.7 Nutzererlebnis und Konsistenz

TKX bietet laut interner Übersicht (`.github/skills/tkx/01-einstieg/uebersicht.md`) ein zentrales Design System mit standardisierten Komponenten und Layoutvorgaben; die technische Dokumentation beschreibt außerdem Validierungsmechanismen. Daraus sind konsistentere Bedienung, bessere Wartbarkeit und weniger individuelle Layoutfehler zu erwarten. Die neue Anwendung verwendet unter anderem `Grid`, `ExpandablePanel`, `TabPanel`, `Combobox`, `DateInput` und `ComponentId`.

Eine tatsächliche Verbesserung der Gebrauchstauglichkeit ist durch den Code allein nicht bewiesen. Dafür wären GUI-Abnahme, Accessibility-Prüfung oder Nutzertests nötig. Die Arbeit sollte zwischen **Frameworkpotenzial** und **empirisch nachgewiesener UX-Wirkung** unterscheiden.

---

## 6. Quantitative Indikatoren mit Einschränkungen

| Indikator | TK Classic | TKX | Aussagegrenze |
|---|---:|---:|---|
| Java-LOC im untersuchten Baum | ca. 8.900 | ca. 5.000 | Näherungswerte; vor Zitation reproduzierbar neu zählen |
| größte zentrale Klasse | Agent ca. 623 LOC | `AdministrationPanel` ca. 327 LOC; `FachdatenDTO` ca. 670 LOC | DTO und Verhaltensklasse sind nur bedingt vergleichbar |
| Nachrichtentyp-Dispatch in der UI-Steuerung | 5 Kaskaden im Legacy-Agent | 0 in der TKX-Formularorchestrierung | UI-Strukturvergleich |
| typabhängige Cast-Zweige | je bis zu 11 in `callInit` und `callUpdate` | 0 im Formularaufbau | kein Gesamtvergleich |
| Nachrichtentyp-Anwendungseinträge | 11 in Legacy-`.ac` | 1 TKX-Anwendung mit Laufzeitauswahl | anderer Navigationsentwurf |
| Anwendungskapsel | 285 Zeilen, 13 Applications insgesamt | 34 Zeilen, 1 Application | Legacy enthält zwei weitere Applications |

> Die Werte sind Momentaufnahmen des untersuchten Repository-Stands. Für die finale TFL sollten Commit-ID, Zählbefehl und Datum dokumentiert werden.

---

## 7. Risiken und Trade-offs

| Entscheidung | Vorteil | Kosten/Risiko |
|---|---|---|
| Strategy je Nachrichtentyp | lokaler Variationspunkt | viele kleine Klassen; Registry bleibt zentral |
| Wiederverwendbare Abschnitte | weniger Duplikation | generische Konstruktoren mit mehreren Booleans können schwer lesbar werden |
| Gemeinsames `FachdatenDTO` | einfacher gemeinsamer Vertrag | God DTO, viele irrelevante Felder je Typ |
| UI-Neuaufbau mit `removeAll` | einfache Konsistenz | potenzieller Zustands- und Performanceverlust |
| DTO statt Business Object in der View | geringere Persistenzkopplung | zusätzlicher Mappingaufwand |
| Presenter-Schicht | testbare Koordination fachlicher Aktionen | zusätzlicher Mapping- und Schnittstellenaufwand |
| Statische Registry | simpel, deterministisch und leicht auffindbar | zentraler Registrierungspunkt, weniger flexibel austauschbar |
| TKX-Standardlayout | Konsistenz und weniger Layoutcode | weniger Freiheit für Speziallayouts |

---

## 8. Bewertbare Hypothesen für die Transferleistung

Aus der Codeanalyse lassen sich folgende Hypothesen ableiten:

- **H1:** Eine explizite Strategy reduziert die Anzahl bestehender UI-Steuerungsklassen, die für einen neuen Nachrichtentyp geändert werden müssen.
- **H2:** Ein gemeinsamer Abschnittsvertrag reduziert typabhängige Casts im Formularaufbau.
- **H3:** TKX-Factory- und Container-APIs reduzieren Layout-Boilerplate gegenüber Guide-Line-basiertem Swing-Code.
- **H4:** Ein Supervising Presenter verbessert die Testbarkeit fachlicher Aktionen, ohne jede lokale Feldänderung durch eine zusätzliche Schicht leiten zu müssen.
- **H5:** TKX allein reduziert die Kopplung nicht automatisch; der größte Effekt entsteht erst durch die Kombination aus Frameworkschnittstellen, Strategy, Abschnittskomposition und Presenter.

Mögliche Prüfung:

1. kontrolliertes Änderungsszenario „zwölfter Nachrichtentyp“,
2. geänderte Dateien und Zeilen zählen,
3. Anzahl Typprüfungen und Casts vergleichen,
4. notwendige Test-Doubles ermitteln,
5. Validierungs- und XML-Parität durch automatisierte Tests prüfen.

---

## 9. Fazit

Die TKX-Zielarchitektur verbessert vor allem **Modularität, lokale Verständlichkeit und Erweiterbarkeit**. Der Fortschritt ist nicht TKX allein zuzuschreiben, sondern der Kombination aus TKX-Komponentenmodell, Strategy, polymorpher Abschnittskomposition, Observer-basiertem Datenfluss und einer klaren Presenter-Rolle.

TKX gibt dabei wesentliche Leitplanken vor: abstrakte Komponenten statt konkreter Widget-Vererbung, uniforme Container, standardisierte Ereignisse, Layoutkonventionen und einen Composition Root. Diese Vorgaben machen den musterorientierten Entwurf wahrscheinlicher, garantieren ihn aber nicht. Die Patterns sind im konkreten Fall sinnvoll, weil sie vorhandene Variabilität und Wiederverwendung strukturieren. Ein unnötig strenges Passive-View-MVP oder ein vollständiges Composite würden dagegen mehr Indirektion erzeugen, als der Formularfall benötigt.
