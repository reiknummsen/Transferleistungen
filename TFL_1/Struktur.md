Hier ist der detaillierte, operative **Strukturplan für deine Transferleistung (TFL)**. 

Dieser Plan legt fest, **WAS** du in welchem Abschnitt schreibst, **WELCHE Methodik** zum Einsatz kommt, **WELCHE Quellen** du benötigst und **WELCHE Fallstricke** du unbedingt vermeiden musst.

> ⚠️ **Wichtiger Grundsatz zum 10-Seiten-Umfang**: Die von der NORDAKADEMIE vorgeschriebenen **10 Seiten (+/- 10%)** beziehen sich rein auf den **fortlaufenden Fließtext** (ca. 3.000 bis 3.500 Wörter). Deckblatt, Verzeichnisse, Abbildungen, Tabellen, Code-Snippets, großer Whitespace und der Anhang zählen **nicht** zu diesen 10 Textseiten. Code gehört in den Anhang oder muss im Text in 1–2 Zeilen synthetisiert werden.

---

### Kapitelübersicht & Gewichtsverteilung

| Kapitel | Seitenumfang (reiner Text) | Schwerpunktthema |
| :--- | :--- | :--- |
| **1. Einleitung** | ca. 1,0 Seite | Problemstellung, Zielsetzung & Forschungsfrage |
| **2. Theoretische Grundlagen** | ca. 1,5 Seiten | GoF-Muster, MVP, ISO/IEC 25010 |
| **3. Methodisches Vorgehen** | ca. 0,75 Seiten | Vergleichende Fallstudie & Kriterienkatalog |
| **4. Fall A: Legacy-Architektur (TK Classic)** | ca. 1,75 Seiten | Ist-Analyse: ViewAgent, God Class, Kaskaden |
| **5. Fall B: Zielarchitektur (TKX)** | ca. 2,25 Seiten | Soll-Analyse: Strategy, Registry, MVP |
| **6. Vergleichende Diskussion & Reflexion**| ca. 2,0 Seiten | Kriterienvergleich, Szenario, Trade-offs |
| **7. Fazit & Handlungsempfehlungen** | ca. 0,75 Seiten | Beantwortung Forschungsfrage & Transfer |

---

### Detaillierter Ausarbeitungsplan

#### 1. Einleitung (ca. 1,0 Seite pure Text)
* **WAS du schreibst**:
  * *Hinführung*: Die Herausforderung langlebiger Enterprise-Software in der gesetzlichen Krankenversicherung (Laufzeiten > 15 Jahre, veraltete UI-Frameworks).
  * *Praxis-Kontext*: Der Datenaustausch Vorsorge/Reha (DA VR) im TKeasy-Produkt *Kur* bei der Techniker Krankenkasse (TK).
  * *Problem*: TK Classic basiert auf Java Swing/ViewAgent. Neue Nachrichtentypen erfordern Änderungen an vielen Stellen (*Wartungsstau*, Verletzung des *Open-Closed-Principle*).
  * *Ziel & Forschungsfrage*: Evaluierung der TKX-Zielarchitektur. *Leitfrage*: „Inwiefern verbessert der Einsatz objektorientierter Entwurfsmuster im TKX-Framework die Erweiterbarkeit und Wartbarkeit der DA-VR-Anwendung im Vergleich zur TK-Classic-Legacy-Architektur?“
* **Methodik**: Deduktive Problemabgrenzung (vom allgemeinen Software-Obsoleszenz-Problem zur spezifischen DA-VR-Architektur).
* **Quellentyp**: Externe Literatur zu Software-Evolution & Legacy-Systemen (z. B. Fowler, Lehman) + TK-Kontextdokumente (`00-ueberblick-und-fragestellung.md`).
* **Fallstricke & Worauf achten**:
  * ❌ *Fehler*: Wie ein Werbeprospekt klingen ("TKX ist das moderne Framework der TK").
  * ✅ *Richtig*: Rein wissenschaftlich-neutral formulieren. Das Problem ist nicht "Swing ist alt", sondern "starke Kopplung und fehlende Kapselung von Variationspunkten erschweren die Wartung".

---

#### 2. Theoretische & Begriffliche Grundlagen (ca. 1,5 Seiten)
* **WAS du schreibst**:
  * *Software-Wartbarkeit*: Definition nach **ISO/IEC 25010** (Fokus auf *Modularität*, *Erweiterbarkeit*, *Analysierbarkeit*, *Prüfbarkeit*).
  * *GoF-Entwurfsmuster*: Wissenschaftliche Definition von *Strategy*, *Registry* (Factory-Variant) und *Observer*.
  * *UI-Architekturmuster*: Abgrenzung von klassischen MVC zu **Model-View-Presenter (MVP)** (speziell *Supervising Controller* / *Presentation Model*).
* **Methodik**: Literaturbasierte Konzeptdefinition (Systematic Literature Review).
* **Quellentyp**: **Ausschließlich externe, zitierfähige Fachliteratur**:
  * Gamma et al. (1994) / Design Patterns (GoF).
  * Fowler (2002/2006) für MVP & Enterprise Patterns.
  * ISO/IEC 25010 Standard für Qualitätskriterien.
* **Fallstricke & Worauf achten**:
  * ❌ *Fehler*: Hier schon Code aus der TK erklären oder interne TK-Dokus zitieren.
  * ✅ *Richtig*: Kapitel 2 liefert **reines Lehrbuchwissen**. Es baut das theoretische Lineal auf, mit dem du in Kapitel 4–6 misst.

---

#### 3. Methodisches Vorgehen: Qualitative Fallstudie (ca. 0,75 Seiten)
* **WAS du schreibst**:
  * *Forschungsdesign*: Begründung der **qualitativen vergleichenden Fallstudie** (*Single Case, Embedded Design*).
  * *Untersuchungsobjekt*: DA-VR-Anwendung im Produkt *Kur*.
  * *Vergleichseinheiten*: **Fall A** (TK Classic Ist-Zustand) vs. **Fall B** (TKX Zielbild).
  * *Operationalisierung*: Ableitung konkreter Indikatoren aus ISO 25010 (Kopplung, Kohäsion, Zyklomatische Komplexität/Kaskadentiefe, Anzahl berührter Klassen bei Erweiterung).
  * *Szenario-Methode*: Beschreibung des kontrollierten Änderungsszenarios (*"Hinzufügen eines 12. Nachrichtentyps"*).
* **Methodik**: Qualitative vergleichende Fallstudien-Methodik (nach Yin / Eisenhardt) & Szenarioanalyse.
* **Quellentyp**: Externe Methodenliteratur (z. B. Wissenschaftliches Arbeiten / Methodenteil, NAK-Leitfäden).
* **Fallstricke & Worauf achten**:
  * ❌ *Fehler*: Nur schreiben "Ich nutze eine Fallstudie".
  * ✅ *Richtig*: Exakt darlegen, *wie* der Quellcode analysiert wurde und wie die Zuordnung von GoF-Mustern zu Codebausteinen erfolgt.

---

#### 4. Fall A: Ist-Analyse der Legacy-Architektur (TK Classic) (ca. 1,75 Seiten)
* **WAS du schreibst**:
  * *Architekturaufbau*: Das ViewAgent-Pattern (Verbindung von Swing-UI, Dateneingabe und Logik).
  * *Befund 1 (God Class)*: Der `DatenaustauschRehaAgent` als zentraler Monolith (623 LOC, 44 Importe, Mischung aus UI-, Mapping- und Ablauflogik).
  * *Befund 2 (Kaskaden & OCP)*: Die 5 typabhängigen `if`/`instanceof`-Kaskaden zur Steuerung von Formularaufbau und Validierung.
  * *Befund 3 (Kopplung)*: Direkte Abhängigkeiten zu domänenspezifischen `Impl`-Klassen anstelle von Interfaces.
* **Methodik**: Empirische Code-Analyse / Statische Strukturanalyse (Kausalanalyse von Wartbarkeitsproblemen).
* **Quellentyp**: **Eigene Analyse / Primärdaten** aus den technischen Projektdokumenten (`01-legacy-architektur.md`, `05-code-snippets.md`).
* **Fallstricke & Worauf achten**:
  * ❌ *Fehler*: Lange Java-Codeblöcke abdrucken, um Seiten zu füllen.
  * ✅ *Richtig*: **Keine Code-Wüsten im Fließtext!** Beschreibe die Struktur präzise in Worten (z. B.: *"Die Steuerung des Importvorgangs erfolgt über eine fünfstufige `instanceof`-Kaskade..."*). Verweise für Details auf den Anhang.

---

#### 5. Fall B: Soll-Analyse der Zielarchitektur (TKX) (ca. 2,25 Seiten)
* **WAS du schreibst**:
  * *Entkopplung durch Strategy*: Kapselung der nachrichtentypspezifischen Logik in `TestnachrichtKonfiguration`-Klassen.
  * *Lookup via Registry*: Entkopplung der Erzeugung durch `TestnachrichtKonfigurationFactory`.
  * *Abschnittskomposition*: Dynamischer UI-Aufbau über `TestnachrichtAbschnitt`.
  * *MVP-Muster*: Trennung der Zuständigkeiten über den `TestnachrichtPresenterImpl` (Supervising Controller) und DTOs.
* **Methodik**: Architectural Pattern Mapping (Zuordnung von GoF-Theorie aus Kap. 2 auf TKX-Klassen).
* **Quellentyp**: **Eigene Analyse / Primärdaten** (`02-tkx-zielarchitektur.md`, `03-design-patterns.md`, `04-mvc-mvp-und-bewertung.md`).
* **Fallstricke & Worauf achten**:
  * ❌ *Fehler (Kritischer Fachfehler!)*: Die Abschnittskomposition in TKX als GoF-*Composite* (Baumstruktur) bezeichnen.
  * ✅ *Richtig*: Stelle klar heraus: TKX nutzt eine **flache Liste polymorpher Abschnitte** mit starkem Operationsvertrag, **kein** tiefes GoF-Composite. Das zeigt deine hohe fachliche Durchdringung!

---

#### 6. Vergleichende Diskussion & Reflexion (ca. 2,0 Seiten)
* **WAS du schreibst**:
  * *Kriterienbasierter Vergleich*: Gegenüberstellung von TK Classic und TKX anhand der ISO-25010-Kriterien (Modularität, Erweiterbarkeit, Testbarkeit).
  * *Änderungsszenario*: Gegenüberstellung des Aufwands beim Hinzufügen eines 12. Nachrichtentyps.
    * *Classic*: Manuelle Anpassung an 5 Kaskaden im Agent, Erhöhung der Komplexität, Regressionsrisiko.
    * *TKX*: Anlegen einer neuen `Strategy`-Klasse, Registrierung in Factory. Bestehender Code bleibt unberührt (Einhaltung des *Open-Closed-Principle*).
  * *Kritische Würdigung & Trade-offs*:
    * Höheres initiales Klassen- und DTO-Volumen (Boilerplate).
    * TKX als *Enabler*: Das Framework liefert die Leitplanken, aber die Entwurfsdisziplin verbleibt beim Entwickler.
* **Methodik**: Synthese, komparative Evaluation & Kritische Reflexion.
* **Quellentyp**: **Reine eigene wissenschaftliche Leistung / Synthese** (Verknüpfung von Kap. 2, 4 und 5).
* **Fallstricke & Worauf achten**:
  * ❌ *Fehler*: Einseitiges Bashing von Classic oder unkritische Lobhudelei auf TKX.
  * ✅ *Richtig*: Sachliche Trade-offs benennen. Zeigen, dass Entkopplung mit einer höheren Anzahl an Indirektionen und Datentransfer-Klassen erkauft wird.

---

#### 7. Fazit & Handlungsempfehlungen (ca. 0,75 Seiten)
* **WAS du schreibst**:
  * *Antwort auf die Forschungsfrage*: Prägnante Zusammenfassung, wie und in welchem Maße TKX durch Entwurfsmuster die Wartbarkeit verbessert.
  * *Praxistransfer / Handlungsempfehlungen*: Was bedeutet das für künftige Migrationen bei der TK? (z. B. Schulung der Entwickler auf Pattern-Disziplin, Erstellung von Architekturlinien für Schnittstellen).
  * *Limitationen*: Abgrenzung der Arbeit (z. B. Fokus auf Architektur, keine Laufzeit- oder Performance-Messungen durchgeführt).
* **Methodik**: Induktive Ableitung von Handlungsempfehlungen aus den Untersuchungsergebnissen.
* **Quellentyp**: **Eigene Arbeit**.
* **Fallstricke & Worauf achten**:
  * ❌ *Fehler*: Neue Argumente oder Quellen einführen, die vorher nicht erwähnt wurden.
  * ✅ *Richtig*: Binde den Sack zu. Die Einleitung verspricht die Antwort, das Fazit liefert sie exakt.

---

### Schnellübersicht: Welcher Abschnitt braucht was?

```
+-----------------------------------------------------------------------------------+
| Kapitel            | Methodik                   | Hauptquellen-Typ              |
+-----------------------------------------------------------------------------------+
| 1. Einleitung      | Deduktion                  | Externe Lit. + TK-Kontext     |
| 2. Grundlagen      | Literatur-Synthese         | REIN Externe Fachliteratur    |
| 3. Methodik        | Qualitative Fallstudie     | Externe Methodenliteratur     |
| 4. Fall A (Classic)| Empirische Code-Analyse    | EIGENE ANALYSE (TK-Code)      |
| 5. Fall B (TKX)    | Pattern Mapping            | EIGENE ANALYSE (TK-Code)      |
| 6. Diskussion      | Szenario-Vergleich         | REINE EIGENE LEISTUNG         |
| 7. Fazit           | Induktion & Transfer       | REINE EIGENE LEISTUNG         |
+-----------------------------------------------------------------------------------+
```

Möchtest du, dass wir als Nächstes die **Auftragsklärung** (die 2.000 Zeichen für die Moodle/CIS-Freigabe der NAK) auf Basis dieses Plans präzise ausformulieren?