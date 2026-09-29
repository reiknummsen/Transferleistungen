# TFL — Überblick, Kontext und Fragestellung

> Autor: Reik Nummsen (P232725, IT.TA.LEVE) · Copilot (2026-09-22)
> Wissensdokument 0 von 7 für die Transferleistung „Migration einer TKeasy-Swing-Anwendung nach TKX unter Einsatz von Entwurfsmustern“

---

## 1. Gegenstand der Untersuchung

Untersuchungsgegenstand ist die **Testnachrichten-Anwendung für den Datenaustausch Vorsorge/Reha (DA VR)** im TKeasy-Produkt **Kur** (Vorsorge/Reha) der Techniker Krankenkasse.

Die Anwendung erlaubt Fachanwendern und Entwicklern, **XML-Testnachrichten** nach dem Standard „Datenaustausch Reha“ (Aufnahme, Entlassung, Rechnung, Unterbrechung, Fehlernachricht, …) manuell zu erzeugen, zu importieren, zu exportieren und in die Verarbeitungsstrecke einzuspeisen. Sie ist damit ein **Testwerkzeug für eine B2B-Schnittstelle** zwischen Krankenkasse und Rehabilitationseinrichtungen.

Es existieren zwei Implementierungen desselben fachlichen Anwendungsfalls:

| | **TK Classic (Legacy)** | **TKX** |
|---|---|---|
| Bundle | `kur.ui.qs` | `testdaten-kur.ui` |
| Package | `de.tk.ui.viewagent.leistung.kur.datenaustausch.testnachricht` | `de.tk.ui.tkx.testdaten.kur.davr.testandwendung` |
| UI-Technologie | Java Swing + TK-GUI-Framework (`TkPanel`, `TkGuideLineLayout`) | TKX (serverseitig programmierte abstrakte UI, Browser-/Electron-Client) |
| Architekturstil | ViewAgent-Pattern (Struct / Agent / View / Panel / Step) | MVP-orientierte UI-Struktur mit DTOs, Presenter und modularen Abschnitten |
| Umfang | 59 Java-Dateien, ca. 8.900 LOC | 65 Java-Dateien, ca. 5.000 LOC |
| Einstiegspunkt | `.ac`-Anwendungskapsel -> `tkeasy_bearbeitung` -> `DatenaustauschRehaStep` | `.ac`-Anwendungskapsel -> `tkx view=...` -> `TestnachrichtUIConstructor` |

Die TKX-Variante wird parallel zur klassischen Anwendung aufgebaut. Diese Koexistenz entspricht einem grundlegenden Gedanken des **Strangler-Fig-Pattern** und bildet den Migrationskontext. Gegenstand der Untersuchung ist jedoch nicht der Projektfortschritt, sondern die architektonische Wirkung des Wechsels von TK Classic zu TKX.

---

## 2. Wissenschaftliche Fragestellung

> **Leitfrage:** Wie unterstützt und prägt TKX gegenüber dem klassischen TKeasy-Frontend (TK Classic/ViewAgent) den Einsatz von Entwurfsmustern, und in welchem Maß verbessert die gezielte Nutzung von *Strategy*, polymorpher Abschnittskomposition, *Observer*, Registry mit factory-artigem Lookup und Model-View-Presenter die Struktur, Erweiterbarkeit und Wartbarkeit der Testnachrichten-Anwendung?

Teilfragen:

1. **Ausgangslage:** Welche Struktur- und Erweiterungsmechanismen bietet TK Classic, und wo erschweren Frameworkkopplung, Vererbung und typabhängige Fallunterscheidungen die Weiterentwicklung?
2. **Framework-Einfluss:** Welche Eigenschaften von TKX – abstrakte Komponenteninterfaces, `UIFactory`, Container, Ereignisschnittstellen, Layoutvorgaben und `ApplicationUIConstructor` – schaffen günstige Voraussetzungen für Entwurfsmuster?
3. **Musteranalyse:** Wie werden *Strategy*, polymorphe Abschnittskomposition, *Observer*, Registry mit factory-artigem Lookup und MVP in der Zielarchitektur eingesetzt?
4. **Nutzenbewertung:** Welche konkreten Probleme lösen diese Muster, und an welchen Stellen wäre ihr Einsatz unnötig komplex oder nur bedingt hilfreich?
5. **Architekturwirkung:** Welche Verbesserungen ergeben sich für Kopplung, Kohäsion, Erweiterbarkeit, Testbarkeit und Konsistenz der Oberfläche?

---

## 3. Vorgeschlagene Gliederung (10 Seiten)

| Kap. | Inhalt | Seiten |
|---|---|---|
| 1 | Einleitung: Ausgangslage, Problemstellung, Zielsetzung, Aufbau | 1 |
| 2 | Grundlagen: TK Classic, TKX, GoF-Entwurfsmuster und MVC/MVP | 1,5 |
| 3 | TK Classic als Vergleichsbasis: Frameworkmodell, vorhandene Muster und strukturelle Grenzen | 2 |
| 4 | TKX als Enabler und Leitplanke: Strategy, Abschnittskomposition, Registry, Observer und MVP | 2,5 |
| 5 | Nutzen und Angemessenheit der Pattern-Implementierung: Erweiterbarkeit, Testbarkeit, Kopplung und Trade-offs | 2 |
| 6 | Fazit und architektonischer Ausblick | 1 |

---

## 4. Dokumentenübersicht dieses Wissenspakets

| Datei | Inhalt |
|---|---|
| `00-ueberblick-und-fragestellung.md` | dieses Dokument: Kontext, Leitfrage, Gliederung |
| `01-legacy-architektur.md` | ViewAgent-Pattern, Struct-Hierarchie, Schwachstellenanalyse |
| `02-tkx-zielarchitektur.md` | TKX-Zielarchitektur, Framework-Fit und Klassenlandkarte |
| `03-design-patterns.md` | Patternanalyse, Nutzenbewertung und Trade-offs — TK Classic vs. TKX |
| `04-mvc-mvp-und-bewertung.md` | MVC/MVP-Vergleich und Bewertung der Architekturqualität |
| `05-code-snippets.md` | kuratierte, zitierfähige Code-Ausschnitte |
| `06-notebooklm-quellen-und-prompts.md` | Importreihenfolge, Quellenstrategie, Methodik und Schreibprompts |

---

## 5. Glossar (Kurzfassung)

| Begriff | Bedeutung |
|---|---|
| **TKeasy** | Modulare Sachbearbeitungsplattform der TK, historisch OSGi-basiert |
| **Kur / Vorsorge-Reha** | TKeasy-Produkt zur Bearbeitung von Vorsorge- und Rehabilitationsleistungen |
| **DA VR** | Subdomäne „Datenaustausch mit Kureinrichtungen“ |
| **TKX** | TK-Eigenentwicklung: serverseitiges UI-Framework mit abstrakten Java-Komponenten; Darstellung im Browser-/Electron-Client |
| **ViewAgent** | Legacy-UI-Architektur der TK: Struct (Daten) + Agent (Logik) + View/Panel (Darstellung) + Step (Workflow) |
| **Struct** | Legacy-Datencontainer aus `ViewAgentValue`-Feldern, gleichzeitig UI-Bauanweisung |
| **`.ac`-Datei** | Anwendungskapsel: XML-Deskriptor, der eine Anwendung im TKeasy-Menübaum registriert |
| **BO** | Business Object (persistentes Fachobjekt, z. B. `KurAufnahmeImpl`) |
| **Abschnitt** | Neuer polymorpher und wiederverwendbarer Baustein eines Nachrichtenformulars (`TestnachrichtAbschnitt`) |
| **Konfiguration** | Neue Strategy, die je Nachrichtentyp die Abschnittsliste festlegt |
| **TK Classic** | Klassisches TKeasy-Frontend auf Basis von Swing, ViewAgent, Structs und TK-GUI-Komponenten |
