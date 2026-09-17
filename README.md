# Elektro-Meister Anki

Dieses Repository ist ein privater Fork von [Anki](https://github.com/ankitects/anki),
dem Open-Source-Programm fuer Spaced-Repetition-Karteikarten. Er dient als
Arbeitsumgebung fuer das Projekt **Elektro-Meister Anki**: eine
Anki-Kartensammlung fuer die Elektro-Meisterpruefung Teil I und II
(Rahmenlehrplan Elektromeister Handwerk - Energie- und Gebaeudetechnik,
HWK Frankfurt (Oder)). Das Projektziel im Detail steht in
[CLAUDE.md](./CLAUDE.md).

Neben dem vollstaendigen Anki-Quellcode enthaelt dieses Repo unter
[templates/elektro-meister/](templates/elektro-meister/) die Karten-
Templates des Projekts. Dieses Dokument beschreibt den aktuellen Stand der
sechs Kartentypen, den Vergleich mit zwei verwandten Projekten
(AnkiCollab-Plugin, anki-eco) und den geplanten Prozess, um Fragen aus dem
Kurs automatisiert zu sammeln, in Karten umzuwandeln und korrekt
einzusortieren. Der Prozess ist ein Entwurf und wird nach Test mit der
Klasse verfeinert.

> Hinweise zum Bauen/Entwickeln des Anki-Quellcodes selbst:
> [CLAUDE.md](./CLAUDE.md) und [docs/development.md](./docs/development.md).
> Lizenz: [LICENSE](./LICENSE) (Anki, AGPL-3.0-or-later). Mitwirkende an
> Anki: [CONTRIBUTORS](./CONTRIBUTORS).

## 1. Kartentypen

| Typ | Ordner | Status |
|---|---|---|
| Einfach (Frage/Antwort) | [templates/elektro-meister/einfach/](templates/elektro-meister/einfach/) | fertig |
| Multiple Choice | [templates/elektro-meister/multiple-choice/](templates/elektro-meister/multiple-choice/) | fertig |
| True/False | [templates/elektro-meister/true-false/](templates/elektro-meister/true-false/) | fertig |
| Drag & Drop (Zuordnung) | [templates/elektro-meister/drag-drop/](templates/elektro-meister/drag-drop/) | fertig, JS-frei (Basis) |
| Bildverdeckung (Image Occlusion) | [templates/elektro-meister/bildverdeckung/](templates/elektro-meister/bildverdeckung/) | fertig, nutzt Ankis nativen Notiztyp |
| Lueckentext (Cloze) | [templates/elektro-meister/luechentext/](templates/elektro-meister/luechentext/) | fertig |

Alle Templates teilen sich ein visuelles System (`--em-*`-CSS-Variablen,
`.em-card`, `.em-kicker`, `.em-answer`, `.em-explanation`), sind dark-mode-
faehig, mobil-optimiert und kommen ohne externe Abhaengigkeiten aus. Details
und Feldbeschreibungen stehen jeweils im README des Unterordners.

## 2. Vergleich mit verwandten Projekten

Im Rahmen dieses Projekts liegen zwei Forks lokal vor (nicht die
oeffentlichen Original-Repos):
`/Users/rimark/Desktop/PlatformIO/anki-eco` und
`/Users/rimark/Desktop/PlatformIO/AnkiCollab-Plugin`.

### anki-eco

anki-eco ist ein Bun/Nx-Monorepo mit fertig gebauten "Classic"-Templates
(MCQ, Cloze, Match, Input, True/False) in React/Preact, inklusive Markdown-,
Mathe- und Mermaid-Unterstuetzung, sowie einer `CardMotion`-Erweiterung fuer
Review-Animationen. Ein `@anki-eco/packager`-CLI baut die Templates zu
`.apkg`-Dateien.

**Entscheidung:** kein Vollumzug des Projekts nach anki-eco. Die einfachen,
statischen Kartentypen (Einfach, MC, True/False, Lueckentext,
Bildverdeckung) bleiben bei ihrem Zero-JS-Ansatz - sie werden per
Copy-Paste in Ankis "Karten..."-Dialog gepflegt, ohne Build-Kette, was fuer
nicht-technische Mitwirkende wichtig ist. Aus anki-eco werden nur gezielt
zwei Techniken uebernommen, **beide bewusst auf eine spaetere Phase
verschoben**, bis der Kern-Prozess (Einreichung -> Agent -> Vorschau ->
AnkiCollab) sauber laeuft:

- **Match-Template als Vorbild** fuer eine spaetere interaktive Version
  unseres [drag-drop/](templates/elektro-meister/drag-drop/)-Templates
  (Tipp-zum-Verbinden statt echtem HTML5-Drag, da Letzteres auf
  Mobile-WebViews unzuverlaessig ist). Bis dahin bleibt die aktuelle
  JS-freie, mental-zuordnende Variante die Basis.
- **CardMotion-Ansatz** als Referenz fuer die animierten SVG-Zeigerdiagramme
  (siehe Abschnitt 4.3) - dort wird gezeigt, wie sich Animationen als
  eigenstaendiges, wiederverwendbares Template-Fragment bauen lassen, das
  auf Desktop, AnkiMobile, AnkiDroid und AnkiWeb gleich funktioniert.
  Animierte Grafiken werden generell erst hinzugefuegt, wenn der
  Einreichungs- und Freigabeprozess ohne sie stabil funktioniert.
- **Test- und Build-Pipeline**: anki-eco testet Templates mit Vitest
  (`templates/classic/tests/*.test.ts`) und versioniert sie per Semver. Fuer
  unsere interaktiveren Templates (Zeigerdiagramm, spaetere Drag-&-Drop-
  Variante) lohnt sich ein aehnlicher, aber schlanker Test-Schritt -
  ausschliesslich als Build-/Testwerkzeug, nicht als Laufzeit-Abhaengigkeit
  in der Karte selbst (Projektvorgabe: moeglichst wenig JavaScript *in der
  Karte*, nicht im Werkzeug).
- **Mehrsprachige `translations/*.json`-Struktur** als Vorlage fuer die in
  CLAUDE.md vorgesehene spaetere englische Uebersetzung des Decks.

### AnkiCollab-Plugin

AnkiCollab ist ein bereits vorhandenes, funktionierendes
Kollaborations-Werkzeug: Mitglieder reichen Aenderungsvorschlaege
(CrowdAnki-basiert) ein, ein Maintainer prueft und bestaetigt sie, danach
erhalten alle Abonnenten die aktualisierte Version des Decks
(`getting_started_maintainer.md` / `getting_started_subscriber.md` im
Plugin-Repo).

Wichtigste Konsequenz fuer unseren Prozess: **Wir muessen keine eigene
Einreichungs-/Freigabe-Infrastruktur bauen.** AnkiCollab deckt genau den
Schritt "strukturierter Kartenvorschlag -> menschliche Pruefung -> Merge
ins geteilte Deck" bereits ab, inklusive Medien-Handling
(`media_manager.py`, `media_optimizer.py`) und Notiztyp-/Tag-Verwaltung, die
zu unserer Subdeck-Konvention (`KW-Dozent-HF-Name-Datum`) passt. Der Teil,
den AnkiCollab **nicht** abdeckt, ist die Umwandlung von rohem,
unstrukturiertem Text (Chat-Nachrichten, Notizen) in eine strukturierte
Notiz - das ist die Luecke, die der in Abschnitt 4 beschriebene Agent
schliesst.

## 3. Einreichungsprozess: Fragen und Antworten aus dem Kurs sammeln

Urspruenglich liefen Einreichungen ueber einen Telegram-Bot (siehe unten
zur Entscheidungsfindung); der ist inzwischen **komplett entfernt**.
Aktueller Kanal ist der **Karten-Tutor** in
[meister-lernplattform-infra/chat-tutor](../meister-lernplattform-infra/chat-tutor/) --
ein Web-Chat, direkt in Moodle eingebettet (kein separates Tool, kein
Bot-Hosting/Wartung noetig). Funktional identisch zum urspruenglichen
Ablauf unten, nur ueber einen anderen Kanal erreichbar.

Damals geprueft wurden vier Optionen fuer den Kanal, ueber den Mitlernende
Fragen, Antworten oder Korrekturen einreichen (Kontext fuer die
Entscheidung, nicht mehr aktueller Stand):

| Option | Vorteile | Nachteile |
|---|---|---|
| **Telegram-Bot** (verworfen) | niedrigste Huerde, spontane Eintraege moeglich, Bot-API gut dokumentiert | Moderationsaufwand, eigenes Bot-Hosting/Wartung, kein direkter Bezug zu Anki-Notiztypen |
| **Formular (Google/MS Forms)** | strukturierte Eingabe (Frage/Antwort/Quelle als getrennte Felder), kein Bot-Hosting noetig | hoehere Huerde als Chat, kein spontanes "kurz reinschreiben", separate Auswertung noetig |
| **GitHub-Issue-Vorlage** | direkt versioniert, an dieses Repo gekoppelt, CI-faehig | fuer Nicht-Techniker ungewohnt, GitHub-Account als Huerde |
| **AnkiCollab "Vorschlag einreichen"** | direkt im finalen Format, kein Zwischenschritt | setzt voraus, dass Einreichende bereits eine fertige Anki-Notiz bauen - genau die Huerde, die wir fuer rohe Fragen/Antworten aus dem Unterricht senken wollen |

**Aktueller Ablauf:** der Karten-Tutor (Chat direkt in Moodle) nimmt rohen
Text/Foto entgegen, wandelt Eintraege in strukturierte Kartenvorschlaege
um und liefert nach Freigabe eine **echte, importierbare .apkg-Datei zum
Download** (mit `genanki`, keine laufende Anki-App/AnkiConnect noetig).
Die Person importiert sie selbst in ihre Anki-App und teilt sie wie
gewohnt ueber **AnkiCollab** mit der Klasse. Damit bleibt die vorhandene,
bereits getestete Freigabe-Infrastruktur von AnkiCollab die einzige
Stelle, an der Inhalte tatsaechlich ins gemeinsame Deck gelangen - der
Tutor selbst schreibt nie direkt ins Deck.

### 3.1 Der mehrstufige Dialog im Chat

Bevor ein Entwurf ueberhaupt als .apkg-Datei rausgeht, fuehrt der Tutor
den Meisterschueler durch mehrere Schritte (implementiert in
[meister-lernplattform-infra/chat-tutor/app/karten_agent.py](../meister-lernplattform-infra/chat-tutor/app/karten_agent.py)):

1. **Upload**: Text oder Foto.
2. **Analyse + Rueckfrage Kartentyp/Fach**: der Agent analysiert kurz und
   schlaegt Kartentyp (Einfach/Lueckentext/Multiple-Choice) und Fach vor -
   der Meisterschueler bestaetigt oder waehlt anders.
3. **Vorschau**: die fertige Karte als Text im Chat.
4. **Rueckfrage Aenderung**: passt es, oder soll etwas ueberarbeitet
   werden? Bei Aenderungswunsch baut der Agent die Karte mit dem Feedback
   neu - beliebig oft wiederholbar.
5. **Freigabe -> .apkg-Download**: erst danach entsteht die Anki-Datei -
   nichts wird automatisch, ungeprueft erzeugt.

Der Zustand haengt (anders als beim fruaeheren Telegram-Bot mit
Thread-Zwang) an einer Chat-Session im Browser, nicht an einem
Nachrichten-Thread - simpler, weil ein Web-Chat pro Tab/Fenster ohnehin
nur eine Konversation gleichzeitig hat.

## 4. Der Kartenerstellungs-Agent

### 4.1 Aufgabe

Der Agent nimmt rohe Einreichungen (Text, Dokument, Foto) entgegen und
fuehrt zwei getrennte, gezielte LLM-Aufrufe aus (siehe Abschnitt 3.1 fuer
den vollen Dialogablauf):

1. **Analyse**: Kurzeinschaetzung des Inhalts, Empfehlung eines Kartentyps
   aus Abschnitt 1 (nicht jede Frage passt zu jedem Typ - siehe Zuordnung
   in 4.2) und eine Bereichs-Vermutung - beides nur Vorschlaege fuer die
   Rueckfragen in 3.1, keine Entscheidung.
2. **Bau**: nach Bestaetigung von Kartentyp und Bereich durch den
   Meisterschueler baut der Agent die eigentliche Karte (Frage, Antwort,
   Erklaerung) und rechnet enthaltene Berechnungen eigenstaendig nach
   (`berechnung_check`). Bei einem Aenderungswunsch (Schritt 5 in 3.1)
   laeuft derselbe Bau-Schritt erneut, diesmal mit dem Feedback als
   Zusatzkontext.
3. Nach finaler Bestaetigung (Schritt 6 in 3.1) ein AnkiCollab-Vorschlag
   zur menschlichen Pruefung (Abschnitt 3) - **noch nicht angebunden**,
   aktuell nur geloggt.
4. animierte SVG-Diagramme (Abschnitt 4.3) - **erst in einer spaeteren
   Ausbaustufe**, nicht Teil des aktuellen Prozesses.

### 4.2 Lernmethodik je Kartentyp

Damit die generierten Fragen tatsaechlich lernwirksam sind, orientiert sich
der Agent an etablierten Prinzipien der Lernpsychologie und ordnet sie den
Kartentypen zu:

| Prinzip | Wirkung | eingesetzt in |
|---|---|---|
| **Testing-Effekt / Active Recall** | aktives Abrufen praegt staerker ein als passives Wiederlesen | alle Typen (Anki-Grundprinzip) |
| **Spaced Repetition** | Wiederholung in wachsenden Abstaenden gegen das Vergessen | FSRS-Scheduler von Anki selbst, typunabhaengig |
| **Dual Coding** | Text + Bild wird staerker erinnert als Text allein | Bildverdeckung, animierte Zeigerdiagramme |
| **Elaboration** | "Warum ist das so?" vertieft das Verstaendnis ueber reines Faktenwissen hinaus | `Erklaerung`-Feld in allen Typen |
| **Interleaving** | gemischtes Ueben verschiedener Themen/Rechenwege verbessert Transferleistung | Subdeck-Mischung durch Ankis Reihenfolge, nicht Aufgabe des Agenten |
| **Worked Examples -> Fading** | erst vorgerechnete Beispiele, dann zunehmend Luecken | Lueckentext fuer Rechenschritte (z. B. Ohmsches Gesetz), von vollstaendig vorgerechnet zu einzelnen Werten verdeckt |
| **Desirable Difficulty** | plausible, nicht triviale Distraktoren erhoehen den Lerneffekt gegenueber Rate-Optionen | Multiple Choice, True/False |
| **Konkretes, eindeutiges Item** | eine pruefbare Aussage pro Karte, keine Verkettungen | True/False, Einfach |
| **Retrieval mit minimalem Kontext-Cue** | Karten duerfen die Antwort nicht schon in der Frageformulierung verraten | agentenseitige Pruefregel vor Veroeffentlichung |

Praktisch heisst das: der Agent bevorzugt **Einfach** und **Lueckentext**
fuer Fakten/Formeln, **Multiple Choice** und **True/False** fuer
Verstaendnisfragen mit Abgrenzung zu haeufigen Fehlvorstellungen,
**Bildverdeckung** fuer alles Bild-/Schaltplanbasierte und **Drag & Drop**
fuer Begriffs-/Definitionszuordnungen.

### 4.3 Animierte SVG-Zeigerdiagramme (spaetere Phase)

> Zurueckgestellt, bis Einreichung -> Agent -> Vorschau -> AnkiCollab ohne
> Animationen sauber funktioniert. Dieser Abschnitt beschreibt den Plan,
> ist aber kein Teil der aktuellen Umsetzung.

Fuer komplexe Zahlen und Blindwiderstaende (induktiv/kapazitiv) generiert
der Agent ein SVG mit x-Achse (Realteil) und y-Achse (Imaginaerteil) sowie
rotierenden Zeigern fuer Spannung/Strom und den Phasenwinkel `phi`.

Wichtig fuer die Projektvorgabe "moeglichst wenig JavaScript": SVG kann
sich **ohne jedes JavaScript** animieren, ueber native
`<animateTransform>`/CSS-`@keyframes`-Rotation auf einer `<g>`-Gruppe. Das
laeuft offline und in allen Anki-Clients (WebKit-basiert), ohne
Laufzeit-Skript - der Agent muss also nur Markup generieren, keine
Interaktionslogik. Ein erster Prototyp ist ueber die Design-Frage in
Abschnitt 5 (SVG-Phasor-Prototyp) geplant, sobald Beispieldaten
(konkrete R/L/C-Werte aus dem Moodle-Stoff) vorliegen.

### 4.4 Trigger: "Gibt es neuen Inhalt?"

**Offen** - der Ursprung neuer Inhalte (Moodle-Export, Mitschriften,
Chat-Nachrichten) ist noch nicht festgelegt. Vorschlaege zur Diskussion:

**Aktuell umgesetzt:** aktiver, expliziter Upload im Karten-Tutor-Chat
(Person schickt bewusst Text/Foto) - kein passives Mitlesen eines
Kurs-Chats noetig, damit erledigt sich die urspruenglich diskutierte
Ereignisgetrieben-ueber-Telegram-Option von selbst (kein Telegram-Kanal
mehr vorhanden). Noch offen, falls zusaetzliche automatische
Quellen gewuenscht sind:

1. **Manueller Befehl/Knopf** durch einen Kursverantwortlichen nach einer
   Unterrichtseinheit, z. B. mit angehaengter Foto-/PDF-Mitschrift.
2. **Zeitgesteuerter Scan** eines geteilten Ordners, in den Mitschriften
   manuell abgelegt werden (kein automatischer Moodle-Zugriff noetig,
   dafuer ein zusaetzlicher manueller Schritt fuer die Klasse).
3. **Moodle-Export/-Scraping als Quelle:** technisch moeglich, aber
   ungeklaert bezueglich Zugriffsrechten/Nutzungsbedingungen der
   HWK-Frankfurt-(Oder)-Moodle-Instanz - **nicht** ohne ausdrueckliche
   Klaerung umsetzen.

## 5. Offene Punkte / naechste Schritte

### Erledigt (erste Iteration im Telegram-Bot, seither entfernt)

Diese Punkte wurden urspruenglich im inzwischen geloeschten
`telegram-bot/` gebaut und validiert; die Erkenntnisse/das Design leben im
aktuellen Karten-Tutor weiter:

- [x] Mehrstufiger Rueckfrage-Workflow (Upload -> Kartentyp/Fach-Rueckfrage
      -> Vorschau -> Aenderungsschleife -> Freigabe) validiert.
- [x] LLM-Agent (Anthropic API) fuer Kartentyp-Empfehlung,
      Bereichs-Vermutung, Feld-Erstellung und Ueberarbeitung mit Feedback
      erprobt - kein Heuristik-Fallback, passt nicht zu einem
      interaktiven Rueckfrage-Workflow.
- [x] Bereichs-Klassifikation mit der **vollstaendigen** Fach-Hierarchie
      (74 Eintraege, alle 5 Teile) aus der "Stapeluebersicht"-Tabelle
      statt erfundener Bereiche validiert.
- [x] Dokument-/Foto-Upload mit Claude Vision (kein OCR) als Startpunkt
      des Workflows validiert.

### Aktueller Stand: Karten-Tutor in meister-lernplattform-infra/chat-tutor

- [x] Web-Chat direkt in Moodle eingebettet (`karten_agent.py`,
      `main.py`, `bereiche.py` - Fach-Hierarchie 1:1 uebernommen).
- [x] Freigegebene Karte wird als echte **.apkg-Datei** (`genanki`, siehe
      `anki_export.py`) zum Download angeboten - kein AnkiConnect mehr
      (braucht eine lokal laufende Anki-App, die auf dem Server nicht
      existiert), kein Moodle-Ordner-Upload (technisch nicht moeglich).
      Die Person importiert selbst und teilt wie gewohnt ueber AnkiCollab.
- [ ] Zwei Nummerierungs-Anomalien aus der Quelltabelle klaeren (siehe
      Kommentar in `bereiche.py`): `1.1.2.1` doppelt vergeben, `4.4`
      sowohl Bereichs- als auch Themen-Nummer.
- [ ] Tages-Limit/Kostenbegrenzung fuer Uploads (gab es im Telegram-Bot
      via `usage.py`) im Chat-Tutor noch nicht nachgebaut.
- [ ] Medien-Anhang (Originalfoto) bei Karten, die ein Bild zum Verstaendnis
      brauchen, noch nicht automatisiert - aktuell manueller Schritt.
- [ ] Session-State in `main.py` ist In-Memory - ein Neustart des Diensts
      verwirft offene Chats.
- [ ] Erste 10 Testkarten pro neuem Kartentyp (True/False, Drag & Drop,
      Bildverdeckung, Lueckentext) mit echtem Moodle-Stoff befuellen und
      fachlich pruefen (Meilenstein 3 in CLAUDE.md) - `anki_export.py`
      unterstuetzt aktuell Einfach/Lueckentext/Multiple-Choice, die
      restlichen Typen fehlen noch im .apkg-Export.
- [ ] Endgueltige Trigger-Quelle fuer 4.4 festlegen, sobald klar ist, ob
      zusaetzliche automatische Quellen (statt nur aktivem Upload)
      gewuenscht sind.

### Spaeter (erst nach stabilem Kernprozess)

- [ ] Interaktive Drag-&-Drop-Variante nach Vorbild von anki-ecos
      Match-Template.
- [ ] Pixelgenaues Vorschau-Rendering (Headless-Browser, z. B. Playwright)
      der echten `front.html`/`styling.css`-Templates.
- [ ] SVG-Zeigerdiagramm-Prototyp (Abschnitt 4.3) mit realen
      R/L/C-Beispielwerten aus dem Moodle-Stoff bauen und in einer Testkarte
      pruefen (im Uebungsaufgaben-Tutor bereits umgesetzt, hier fuer
      Anki-Karten noch offen).
