# LinkedIn-Postserie: Phase 3 (Public Launch)

Stand: 13.11.2026 · Status: Entwurf · Grundlage: [Kommunikationsstrategie](Kommunikationsstrategie.md)

## Überblick

| Post | Zeitpunkt | Ziel | Zielgruppe | Format |
|---|---|---|---|---|
| 1 | Tag 1 | Launch, Aufmerksamkeit, Repo-Link | A, B, C, F | Video (3–5 Min) oder Kurzclip |
| 2 | Tag 2–3 | Fachartikel/Blog bekannt machen, Einblick in die Arbeitsweise | A, D | Link-Post mit Bild |
| 3 | Tag 4 | Vorher/Nachher mit Ergebnisbeispiel | A, B, C | Bild-Karussell (Dokument-PDF) |
| 4 | Tag 6–7 | Testaufruf für DMN und Skripte, Zwischenbilanz | A, B | Text, optional Screenshot |

## Regeln für alle Posts

- Pflichtbotschaft in jedem Post: **Entwurf, Prüfung, Test** und **kein offizielles d.velop-Produkt**.
- Nur synthetische Daten in Screenshots und Video.
- Repo-Link nicht im Haupttext, sondern im ersten Kommentar (LinkedIn priorisiert Posts ohne externe Links). Im Post steht dann „Link im ersten Kommentar“. Alternative: Link im Text, falls die Reichweite zweitrangig ist.
- 3–5 Hashtags am Ende, immer `#dvelop #BPMN`.
- Antworten auf Kommentare innerhalb von 24 Stunden, in den ersten 2 Stunden nach Veröffentlichung besonders aktiv.
- Vor Veröffentlichung klären (siehe Strategie, Abschnitt 11): Absender (Privatperson oder Organisation) und Information an d.velop.
- Platzhalter in `[eckigen Klammern]` vor dem Posten ersetzen.

---

## Post 1: Launch (Tag 1)

**Format:** Demo-Video (Anforderung → BPMN → Formular → Import), Untertitel einblenden, da LinkedIn-Videos meist ohne Ton laufen.

**Text:**

> Wie lange dauert es, bis aus einer Prozessbeschreibung ein importierbares Modell in d.velop Process Studio wird?
>
> Ich habe daran gearbeitet, diesen Weg zu verkürzen, und veröffentliche heute den **Artifact Generator for d.velop Process Studio**.
>
> Was dahintersteckt:
> 🔹 GitHub Copilot in VS Code mit spezialisierten Agenten
> 🔹 Vorlagen aus echten Process-Studio-Exporten
> 🔹 Vier Artefakte: BPMN-Prozesse, Formulare, DMN-Tabellen, Skripte
> 🔹 Formulare werden per `camunda:formKey` direkt mit den Benutzeraufgaben verknüpft
>
> Im Video: Aus einer Fachbeschreibung entstehen Prozess und Formular, beides bereit zum Import.
>
> Wichtig für die Einordnung:
> ✅ Open Source (MIT)
> ✅ Eigenständiges Community-Projekt, kein offizielles d.velop-Produkt
> ✅ Die Ergebnisse sind Entwürfe. Jeder Schritt wird bestätigt, geprüft und im Zielsystem getestet.
>
> Das Projekt führt den Ansatz von GovModeler weiter, mit Fokus auf Verwaltungsprozesse.
>
> Ihr arbeitet mit d.velop Process Studio? Probiert es aus und sagt mir, was fehlt.
>
> Link im ersten Kommentar 👇
>
> #dvelop #BPMN #GitHubCopilot #Prozessautomatisierung #eGovernment

**Erster Kommentar:**

> Repo: https://github.com/u103059/artifact-generator-for-dvelop-process-studio
> Quick Start in 5 Schritten direkt im README. Voraussetzung: VS Code, GitHub Copilot und eine Process-Studio-Testumgebung.

**Anschlussaktionen:** Personen aus der Pilotphase und Multiplikatoren (Partner, GovModeler-Kontakte) persönlich per Nachricht auf den Post hinweisen und um Reaktion bitten.

---

## Post 2: Einblick in die Arbeitsweise (Tag 2–3)

**Anlass:** Fachartikel/Blogbeitrag „Von der Prozessbeschreibung zur BPMN-Datei mit Copilot“.

**Text:**

> KI erzeugt kein fertiges Prozessmodell auf Knopfdruck. Und das ist gut so.
>
> Beim Artifact Generator for d.velop Process Studio läuft jeder Auftrag in fünf Schritten:
>
> 1️⃣ Anforderung beschreiben (Prozess, Beteiligte, Daten)
> 2️⃣ Fachlichen Entwurf prüfen und bestätigen
> 3️⃣ Datei anhand der Vorlage erstellen lassen
> 4️⃣ Ergebnis prüfen (Struktur, Referenzen, Variablen, Logik)
> 5️⃣ Im Zielsystem testen
>
> Der Agent fragt bei unvollständigen Angaben nach, statt Alternativpfade zu erfinden. Gateways, Timer oder Eskalationen entstehen nur, wenn die Prozessbeschreibung sie hergibt.
>
> Warum das wichtig ist: Gerade in der Verwaltung braucht es Nachvollziehbarkeit. Ein Entwurf, den ich Schritt für Schritt prüfen kann, ist mehr wert als eine Blackbox.
>
> Im neuen Artikel beschreibe ich den Weg von der Prozessbeschreibung zur BPMN-Datei, mit Beispielen und den Grenzen des Ansatzes: [Link zum Artikel]
>
> Wie geht ihr heute vom Fachkonzept zum Prozessmodell?
>
> #dvelop #BPMN #GitHubCopilot #KIinderVerwaltung #Prozessmanagement

**Bild:** Screenshot des BPMN-Modells „Telearbeit vereinbaren“ ([Screenshot](../examples/example-bpmn-processes/Screenshot%202026-11-13%20112224.png)).

**Zusätzlich an Tag 2–3 (ohne eigenen Post-Text, angepasste Kurzfassung):** d.velop-Community, XING-/Fachgruppen zur Verwaltungsdigitalisierung, GovModeler-Netzwerk.

---

## Post 3: Vorher/Nachher (Tag 4)

**Format:** Karussell (PDF, 4–5 Folien), jeweils ein Gedanke pro Folie.

**Folienvorschlag:**

1. **Titel:** „Von der Fachbeschreibung zum Formular in d.velop Process Studio“
2. **Vorher:** Textbeschreibung des Anliegens (z. B. „Anmeldung an weiterführender Schule: Angaben zum Kind, Sorgeberechtigten, Wunschschule, Nachweise, Einwilligungen“)
3. **Auftrag an den Agenten:** Auszug aus dem Prompt
4. **Nachher:** Screenshot des Formulars ([Screenshot](../examples/example-forms/Screenshot%202026-11-13%20112725.png)) mit Abschnitten, Pflichtfeldern, Auswahlfeld, Einwilligungen
5. **Einordnung:** „Entwurf. Geprüft, importiert und getestet wird in der eigenen Umgebung.“ plus Repo-Hinweis

**Text:**

> Vorher: eine Fachbeschreibung in Textform.
> Nachher: ein Formular mit Abschnitten, Pflichtfeldern, Auswahlfeld und Einwilligungen, geladen in d.velop Process Studio.
>
> Das Beispiel „Anmeldung an weiterführender Schule“ liegt komplett im Repository, ebenso der Prozess „Telearbeit vereinbaren“ und ein Skript zur Fristberechnung. Alles mit synthetischen Daten, zum Anschauen und Nachbauen.
>
> Was ich dabei gelernt habe: Die Qualität hängt stark an der Vorlage. Deshalb basieren die Agenten auf echten Exportdateien aus Process Studio und nicht auf Annahmen.
>
> Ehrlich dazu: Die Ergebnisse sind Entwürfe, die ihr prüfen und im Zielsystem testen müsst. Das Projekt ist ein Community-Projekt, kein offizielles d.velop-Produkt.
>
> Welcher Prozess aus eurem Alltag wäre ein guter nächster Testfall? Schreibt es in die Kommentare, ich nehme Vorschläge als Beispielaufträge auf.
>
> Link im ersten Kommentar 👇
>
> #dvelop #BPMN #Formulare #eGovernment #Prozessautomatisierung

---

## Post 4: Testaufruf und Zwischenbilanz (Tag 6–7)

**Text:**

> Eine Woche nach dem Start des Artifact Generator for d.velop Process Studio: Danke für [Anzahl] Rückmeldungen, [Anzahl] Stars und die Gespräche mit [Zielgruppe, z. B. Partnern und Verwaltungsteams].
>
> Was sich gezeigt hat: [1–2 konkrete Erkenntnisse aus dem Feedback].
>
> Wo ich eure Hilfe brauche:
> 🔍 **DMN-Entscheidungstabellen:** Funktioniert der Import in eurer Umgebung, auch in Verbindung mit einer Geschäftsregelaktivität im BPMN?
> 🔍 **Skripte:** Lassen sich die erzeugten JSON-Dateien über die Importfunktion der Skriptverwaltung einlesen? Werden IDs korrekt übernommen?
>
> Das sind die Stellen, an denen das Projekt bewusst Annahmen kennzeichnet. Eine kurze Rückmeldung als Issue oder Discussion im Repo hilft allen, die danach starten.
>
> Wenn ihr Process Studio nutzt und 30 Minuten für einen Test habt: Meldet euch gern direkt bei mir.
>
> Link im ersten Kommentar 👇
>
> #dvelop #BPMN #DMN #OpenSource #Prozessautomatisierung

**Erster Kommentar:**

> Repo: https://github.com/u103059/artifact-generator-for-dvelop-process-studio
> Ergebnisse von Importtests bitte als Issue oder Discussion melden. Auch „funktioniert nicht“ ist eine wertvolle Rückmeldung.

---

## Optionale Zusatzposts (bei Resonanz)

- **Kurzclip Skript:** 60 Sekunden, Skript „Frist berechnen“ vom Auftrag bis zur Exportdatei.
- **GovModeler-Bezug:** Wie der Ansatz entstanden ist und was sich bei der Weiterentwicklung geändert hat (Zielgruppe C).
- **Danke-Post:** Namentlich (mit Zustimmung) bei Testern und Multiplikatoren bedanken. Vorbereitung auf Phase 4.

## Erfolgskontrolle der Serie

| Kennzahl | Quelle | Richtwert (aus Strategie) |
|---|---|---|
| Rückmeldungen von Anwender:innen/Partnern | Kommentare, Nachrichten, Issues | 5 in 3 Monaten |
| Importtest-Rückmeldungen | Issues, Discussions | 3 in 3 Monaten |
| Stars, Clones, Views | GitHub Insights | 25–50 Stars in 3 Monaten |
| Impressionen, Reaktionen, Klicks, Videoaufrufe | LinkedIn Analytics | Vergleich der vier Posts |
| Multiplikatoren, die teilen | LinkedIn, Community | 2 |

Nach Tag 7 auswerten: Welcher Post erzeugte die meisten qualifizierten Rückmeldungen? Das Ergebnis fließt in den Rhythmus von Phase 4 ein.
