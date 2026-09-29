# NotebookLM-Leitfaden, Quellenbasis und Schreibprompts

> Autor: Copilot (2026-09-22)
> Wissensdokument 6 von 7 · Zweck: Überführung der Codeanalyse in eine zehnseitige wissenschaftliche Transferleistung

---

## 1. Empfohlene Importreihenfolge

Diese Dateien gemeinsam als Quellen in NotebookLM importieren:

1. `00-ueberblick-und-fragestellung.md`
2. `01-legacy-architektur.md`
3. `02-tkx-zielarchitektur.md`
4. `03-design-patterns.md`
5. `04-mvc-mvp-und-bewertung.md`
6. `05-code-snippets.md`
7. `06-notebooklm-quellen-und-prompts.md`

Zusätzlich sollten die hochschulspezifischen Vorgaben, Bewertungsrubrik und Zitierregeln importiert werden. Falls verfügbar, sind Architekturdiagramme oder Screenshots beider Oberflächen sinnvolle Ergänzungen; sie ersetzen jedoch keine Usability-Untersuchung.

### 1.1 Zeichencodierung und Export

Alle Dateien dieses Wissenspakets verwenden **UTF-8** und enthalten echte Umlaute (`ä`, `ö`, `ü`, `Ä`, `Ö`, `Ü`, `ß`). Diagramme verwenden möglichst portable ASCII-Zeichen. Beim Export ist Folgendes zu beachten:

1. Im Editor die Dateikodierung explizit auf **UTF-8** setzen.
2. Nicht als Windows-1252, ANSI oder ISO-8859-1 neu speichern.
3. Beim Kopieren in ein anderes System keine automatische „ANSI“-Konvertierung verwenden.
4. Nach dem Export stichprobenartig nach `für`, `Änderung`, `größer` und `Straße` suchen.
5. Das Unicode-Ersatzzeichen `U+FFFD` oder als Windows-1252 fehlinterpretierte UTF-8-Bytefolgen weisen auf eine fehlerhafte Dekodierung hin.

UTF-8 ohne BOM ist für Markdown grundsätzlich die portabelste Variante. Sollte ein älteres Windows-Werkzeug die Kodierung nicht automatisch erkennen, kann alternativ UTF-8 **mit BOM** exportiert werden; NotebookLM und moderne Markdown-Editoren benötigen den BOM üblicherweise nicht.

---

## 2. Quellenhierarchie

NotebookLM soll Aussagen nach ihrer Herkunft trennen:

### A. Primärquellen aus dem Repository

Sie belegen den tatsächlichen Implementierungsstand:

- Legacy-Code unter `kur.ui.qs/src/de/tk/ui/viewagent/leistung/kur/datenaustausch/testnachricht`
- TKX-Code unter `testdaten-kur.ui/src/de/tk/ui/tkx/testdaten/kur/davr/testandwendung`
- `.ac`- und `.wi`-Deskriptoren
- Projektvorgaben aus `.github/copilot-instructions.md`
- TKX-interne Dokumentation aus `.github/skills/tkx/`
- Architekturentscheidungen unter `docs/architektur/`

### B. Wissenschaftliche und fachliche Sekundärquellen

Sie definieren Muster und Qualitätsbegriffe. Geeignete Ausgangspunkte:

1. **Gamma, Helm, Johnson, Vlissides:** *Design Patterns: Elements of Reusable Object-Oriented Software*. Für Strategy, Composite, Observer, Factory Method und Template Method.
2. **Martin Fowler:** *Patterns of Enterprise Application Architecture*. Für Presentation Model und Schichtungsfragen.
3. **Martin Fowler:** Beschreibungen von *Passive View*, *Supervising Controller* und *Strangler Fig Application* auf martinfowler.com. Abrufdatum dokumentieren.
4. **Robert C. Martin:** *Agile Software Development, Principles, Patterns, and Practices* oder eine geeignete Primärquelle zu SOLID. Für SRP, OCP und Dependency Inversion.
5. **ISO/IEC 25010** beziehungsweise die an der Hochschule verfügbare Fassung. Für Wartbarkeit, Modularität, Analysierbarkeit, Modifizierbarkeit und Testbarkeit.
6. Eine wissenschaftliche Quelle zu Softwaremodernisierung oder Legacy-System-Migration, vorzugsweise aus IEEE Xplore, ACM Digital Library oder SpringerLink.
7. Optional eine wissenschaftliche Quelle zu Design Systems und UI-Konsistenz; die interne TKX-Dokumentation allein ist keine unabhängige Evidenz für UX-Verbesserungen.

> Bibliografische Daten, Ausgabe, DOI/URL und Seitenzahlen müssen vor Abgabe anhand der tatsächlich verwendeten Ausgabe verifiziert werden. Dieses Wissenspaket enthält bewusst keine erfundenen Seitenangaben.

### C. Eigene Analyse

Statische Zählungen, Vergleiche und Bewertungen sind als eigene Untersuchung auszuweisen, zum Beispiel:

> Eigene Darstellung auf Basis einer statischen Analyse des Repository-Stands vom 22.09.2026.

Falls möglich, zusätzlich Commit-ID beziehungsweise Branch-Zustand dokumentieren. Dadurch werden Metriken reproduzierbar.

---

## 3. Welche Aussage benötigt welche Quelle?

| Aussage | geeignete Belegart |
|---|---|
| Definition Strategy/Composite/Observer | GoF-Literatur |
| Abgrenzung MVP, Passive View, Supervising Controller | Fowler/Potel oder wissenschaftliche Architekturliteratur |
| Definition Wartbarkeit/Testbarkeit | ISO/IEC 25010 oder wissenschaftliche Qualitätsliteratur |
| TKX arbeitet serverseitig mit abstrakten UI-Elementen | interne TKX-Dokumentation |
| Agent enthält fünf Typkaskaden | Repository-Code / eigene Analyse |
| Presenter trennt UI-Koordination von Anwendungsfällen | MVP-Literatur plus Klassenrollen der Zielarchitektur |
| Strategy reduziert typabhängige UI-Steuerung | Repository-Code / eigene vergleichende Analyse |
| TKX verbessert tatsächliche Usability | nur mit empirischen Daten; aus Code nicht belegbar |
| Strangler Fig ist vollständig umgesetzt | derzeit nicht belegbar; nur parallele Koexistenz ist sichtbar |

---

## 4. Empfohlene Methodik der Arbeit

### 4.1 Forschungsdesign

Eine **qualitative vergleichende Fallstudie** mit ergänzenden statischen Kennzahlen passt zum Material:

1. Fall und Systemgrenze definieren.
2. Legacy- und TKX-Code anhand derselben Kategorien untersuchen.
3. Patternrollen aus Literatur operationalisieren.
4. Klassen und Methoden den Rollen zuordnen.
5. Auswirkungen auf Qualitätsmerkmale argumentieren.
6. Nutzen, Trade-offs und mögliche Übermodellierung der Patterns abwägen.

### 4.2 Analysekategorien

- Verantwortungsverteilung
- Richtung und Art der Abhängigkeiten
- explizite Variationspunkte
- Polymorphie versus Typabfragen
- Wiederverwendung von UI-Bausteinen
- Datenfluss und Ereignismodell
- Testisolierbarkeit
- Anzahl geänderter Stellen beim Erweiterungsszenario
- Grad der Frameworkunterstützung und Grad der bewussten Anwendungsentscheidung

### 4.3 Kontrolliertes Änderungsszenario

Als Gedankenexperiment oder praktische Mini-Studie kann ein neuer Nachrichtentyp „X“ betrachtet werden:

**Legacy:** Welche bestehenden Methoden und Deskriptoren müssten geändert werden? Welche Casts entstehen?

**TKX:** Welche neue Konfiguration und Abschnitte sind nötig? Muss die Orchestrierung geändert werden? Welche Änderungen entstehen dennoch in Registry, DTO und Mapping?

Dieses Szenario ist aussagekräftiger als LOC allein, weil es die Änderbarkeit der beiden Architekturansätze direkt untersucht.

### 4.4 Limitationen

In der Arbeit explizit nennen:

- nur ein Anwendungsausschnitt und ein Repository-Stand,
- statische Analyse ohne Laufzeitmessung,
- keine Entwicklerinterviews,
- keine Nutzertests oder Accessibility-Messungen,
- interne Frameworkbesonderheiten begrenzen Übertragbarkeit,
- Codeumfang zwischen Alt und Neu ist nicht unmittelbar vergleichbar.

Die Arbeit betrachtet die TKX-Lösung als **konzeptionell abgeschlossene Zielarchitektur**. Zeitliche Details einzelner Migrationsschritte sind nicht Teil der Leitfrage. Konkrete Aussagen über vorhandene Klassen und Methoden müssen trotzdem mit dem Repository übereinstimmen.

---

## 5. Empfohlene Gliederung für zehn Textseiten

| Kapitel | Inhalt | Richtwert |
|---|---|---:|
| 1. Einleitung | Problem, Ziel, Leitfrage, Fallauswahl | 1,0 Seite |
| 2. Grundlagen | TKX/ViewAgent, GoF-Muster, MVC/MVP, Qualitätsmodell | 1,5 Seiten |
| 3. Methodik | Fallstudie, Quellen, Kategorien, Grenzen | 0,75 Seite |
| 4. Legacy-Analyse | Struct-Baum, Template Method, Observer, God Class, fünf Kaskaden | 1,75 Seiten |
| 5. TKX-Analyse | TKX als Enabler/Leitplanke, Strategy, Registry, Abschnittskomposition, Observer und MVP | 2,25 Seiten |
| 6. Vergleich und Diskussion | Pattern-Nutzen, Framework-Fit, Qualitätswirkung und Übermodellierungsrisiken | 2,0 Seiten |
| 7. Fazit und Ausblick | Antwort auf Leitfrage, Ports/Adapter, Tests | 0,75 Seite |

Die genaue Seitenverteilung ist an Layout und Hochschulvorgaben anzupassen.

---

## 6. Direkt nutzbarer Hauptprompt für NotebookLM

```text
Erstelle auf Basis der importierten Quellen einen wissenschaftlichen Rohentwurf für eine
zehnseitige Transferleistung in deutscher Sprache.

Leitfrage:
Wie unterstützt und prägt TKX gegenüber dem klassischen TKeasy-Frontend
(TK Classic/ViewAgent) den Einsatz von Entwurfsmustern, und in welchem Maß verbessert
die konkrete Nutzung von Strategy, polymorpher Abschnittskomposition, Observer,
Registry/Simple Factory und Model-View-Presenter die Struktur, Erweiterbarkeit und
Wartbarkeit der Testnachrichten-Anwendung?

Anforderungen:
1. Trenne Definitionen aus Fachliteratur von Befunden aus dem Repository.
2. Bezeichne TestnachrichtKonfiguration als Strategy.
3. Behaupte nicht, TestnachrichtAbschnitt sei ein vollständiges GoF-Composite. Erkläre
   stattdessen die polymorphe Abschnittskomposition und vergleiche sie mit dem echten,
   aber operational schwachen Legacy-Struct-Baum.
4. Bezeichne TestnachrichtKonfigurationFactory als statische Registry mit zentraler
   Instanziierung und factory-artigem Lookup, nicht als GoF Factory Method.
5. Betrachte die TKX-Struktur als konzeptionell abgeschlossene MVP-orientierte
   Zielarchitektur. Der zeitliche Projektfortschritt einzelner technischer Anbindungen
   ist nicht Gegenstand der Arbeit.
6. Analysiere ausführlich, wie UIFactory, Komponenteninterfaces, Container,
   Ereignisschnittstellen, ApplicationUIConstructor, ComponentId und Layoutvorgaben
   den Pattern-Einsatz ermöglichen oder in eine bestimmte Richtung lenken.
7. Stelle klar, dass TKX die Muster begünstigt und teilweise strukturell nahelegt,
   aber anwendungsspezifische Patterns nicht automatisch erzeugt.
8. Bewerte für jedes Muster Problembezug, Nutzen, zusätzliche Komplexität und
   Angemessenheit. Erkläre auch, warum ein vollständiges GoF-Composite hier nicht
   erforderlich ist.
9. Vergleiche TK Classic fair: Observer, Composite und Template Method existieren
   bereits, sind aber stärker an ViewAgent, Structs und Vererbung gebunden.
10. Verwende die Codebelege aus 05-code-snippets.md und nenne Klassen und Methoden.
11. Erfinde keine Quellen, Seitenzahlen, Messwerte, Frameworkfunktionen oder
   empirischen UX-Ergebnisse.
12. Formuliere analytisch und abwägend, nicht werblich, und beantworte die Leitfrage
   im Fazit ausdrücklich.
```

---

## 7. Prompts für einzelne Arbeitsschritte

### Literaturmatrix

```text
Erstelle eine Literaturmatrix mit den Spalten Quelle, Begriff/Muster, Definition,
verwendbare These, Abgrenzung und vorgesehenes Kapitel. Verwende nur tatsächlich
importierte Literatur und markiere fehlende bibliografische Angaben.
```

### Pattern-Prüfung

```text
Prüfe jede behauptete Pattern-Instanz gegen die Rollen der Originaldefinition.
Gib für jede Rolle die konkrete Klasse/Methode an. Ordne das Ergebnis als vollständig,
teilweise oder nicht erfüllt ein. Vermeide die Ableitung eines Musters allein aus
Klassennamen.
```

### Kritische Diskussion

```text
Formuliere eine Gegenposition zur These, dass mehr Entwurfsmuster automatisch bessere
Software erzeugen. Prüfe zusätzliche Klassen, Indirektion, statische Registry, breites
FachdatenDTO, manuelle Synchronisation und mögliche Übermodellierung. Stelle diesen
Kosten die reale Zahl von Nachrichtentypen, Wiederverwendung der Abschnitte und
Entkopplung der Orchestrierung gegenüber. Führe danach eine abgewogene Synthese durch.
```

### Plagiats- und Evidenzkontrolle

```text
Markiere in diesem Kapitel jeden Satz als Literaturthese, Repository-Befund,
eigene Interpretation oder unbelegte Behauptung. Schlage für unbelegte Behauptungen
eine Quelle, vorsichtigere Formulierung oder Streichung vor.
```

### Kürzung auf zehn Seiten

```text
Kürze den Text, ohne Leitfrage, Methodik, zentrale Codebelege, Gegenargumente und
Limitationen zu entfernen. Streiche zuerst Wiederholungen und reine Klassenauflistungen.
Behalte die Gegenüberstellung der fünf Legacy-Kaskaden mit Strategy/Registry sowie die
Abgrenzung des Composite-Begriffs bei.
```

---

## 8. Formulierungen, die vermieden werden sollten

| Nicht verwenden | Besser |
|---|---|
| „TKX ist deklarativ.“ | „TKX verwendet eine programmatische Java-API mit deklarativen Komponentenmetadaten und Fluent-Konfiguration.“ |
| „TKX kapselt Swing.“ | „TKX beschreibt die UI serverseitig und stellt sie über einen Web-/Electron-Client dar.“ |
| „Composite wurde neu eingeführt.“ | „Das Legacy besitzt einen Struct-Baum; TKX führt einen stärkeren gemeinsamen Abschnittsvertrag und Composite-artige UI-Komposition ein.“ |
| „Die Factory implementiert Factory Method.“ | „Die Klasse ist eine statische Registry mit zentraler Instanziierung und factory-artigem Lookup.“ |
| „Die Anwendung ist automatisch MVP, weil eine Presenter-Klasse existiert.“ | „Die Zielarchitektur ordnet übergreifende UI-Aktionen dem Presenter zu und lässt lokale DTO-Synchronisation bewusst in den Abschnitten.“ |
| „Das OCP ist erfüllt.“ | „Die UI-Orchestrierung ist gegenüber neuen Konfigurationen weitgehend geschlossen; Registry, DTO und Mapping bleiben Änderungspunkte.“ |
| „TKX macht die Anwendung automatisch testbar.“ | „Abstrakte Komponenten, `ComponentId`, DTOs und kleine Verträge schaffen testbare Grenzen; deren Nutzen hängt von der konkreten Zerlegung ab.“ |
| „Die UX ist besser.“ | „TKX verspricht Konsistenz und standardisierte Interaktion; eine UX-Verbesserung wurde nicht empirisch geprüft.“ |
| „Die neue Anwendung ist wegen weniger LOC automatisch besser.“ | „LOC sind ergänzend; entscheidender sind Variationspunkte, Abhängigkeiten und Änderungsumfang.“ |
| „Strangler Fig ist umgesetzt.“ | „Parallele Koexistenz als ein Baustein des Strangler-Ansatzes ist sichtbar.“ |

---

## 9. Abschlusscheck vor Abgabe

- [ ] Jede Patternbezeichnung gegen die Literaturdefinition geprüft
- [ ] Repository-Befunde mit Klassen und Methoden belegt
- [ ] Eigene Interpretationen sprachlich kenntlich gemacht
- [ ] Keine erfundenen Seitenzahlen oder Quellen
- [ ] Commit-ID/Stand und Analysedatum angegeben
- [ ] Fünf Legacy-Kaskaden statt vier genannt
- [ ] Composite und MVP nicht überbehauptet
- [ ] TKX als Enabler und Leitplanke, nicht als automatische Pattern-Lösung dargestellt
- [ ] Nutzen und zusätzliche Komplexität jedes zentralen Patterns bewertet
- [ ] Konzeptionelle Zielbetrachtung klar von konkreten Codebelegen getrennt
- [ ] LOC nur als eingeschränkter Indikator verwendet
- [ ] Leitfrage im Fazit ausdrücklich beantwortet
- [ ] Literaturverzeichnis formal nach Hochschulvorgabe erstellt
- [ ] Datenschutz und interne Vertraulichkeit vor Upload in externe Systeme geprüft

> **Datenschutzhinweis:** Vor dem Upload in NotebookLM muss geprüft werden, ob interne Klassennamen, Projektdetails und Quellcodeausschnitte gemäß den geltenden Unternehmensrichtlinien in das verwendete System übertragen werden dürfen. Dieses Dokument trifft keine Freigabeentscheidung.
