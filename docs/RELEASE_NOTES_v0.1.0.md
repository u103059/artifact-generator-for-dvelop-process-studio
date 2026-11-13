# Release Notes v0.1.0 (Pre-Release)

Erster öffentlicher Release. Von der Fachbeschreibung zur importierbaren d.velop-Process-Studio-Datei – mit KI-Agenten (GitHub Copilot) und geprüften Vorlagen.

> Eigenständiges Community-Projekt (MIT-Lizenz), kein offizielles d.velop-Produkt.

## Enthalten

- **Vier Agenten** in `.github/agents/`: BPMN, DMN, Formulare und Skripte für d.velop Process Studio.
- **Vorlagen** in `Templates/`: `BPMN_Template.xml`, `DMN.Template.md`, `Form.Template.md`, `Script.Template.md`. Features sind mit **[V]** (verifiziert) bzw. **[U]** (ungetestet) gekennzeichnet.
- **Beispiele** in `examples/`: BPMN-Prozesse, Formulare, ein DMN-Beispiel und ein Skript-Beispiel (nur synthetische Daten).
- **Quick Start** mit 5 Schritten in der README.

## Bekannte Grenzen

Alle Ergebnisse sind **Entwürfe** und müssen im Zielsystem importiert und geprüft werden.

### DMN

- Verifiziert ist nur eine einfache Entscheidungstabelle (siehe `examples/example-DMN/`).
- Ungetestet:
  - mehrere Eingabe- und Ausgabespalten
  - Hit Policies außer `UNIQUE`
  - Zahlenvergleiche und Bereiche (`< 1000`, `[1000..5000]`)
  - Alternativen und Negation (`"a","b"`, `not("a")`)
  - Datentypen `Number` und `Boolean`
  - Identity als Eingabe
- Bei überschneidenden Regeln unter `UNIQUE` ggf. `FIRST` verwenden (ungetestet).

### Skripte

- Generierte Exportdateien enthalten keine externen Bibliotheken; bei Bedarf nach dem Import im Tab „Externe Bibliotheken“ ergänzen.
- Ungetestet:
  - Übernahme von `scriptId`, `versionId`, `author` und `editor` beim Import (`author`/`editor` bleiben leer)
  - Rückgabe über `output` und Zuordnung in der Service-Aufgabe
  - Übergabe von Identity-Variablen, Gruppen-URIs und Berechtigungen des Aufrufers
- Nicht freigegebene Skripte werden als `<Name>_draft.json` exportiert.

### Allgemein

- Die Qualität der Ergebnisse hängt vom verwendeten KI-Modell und der Fachbeschreibung ab. Jeden Schritt bestätigen und prüfen.

## Feedback gesucht

Bitte Importtests in der Zielumgebung melden (Zielversion, was funktioniert hat, was nicht), insbesondere für die oben genannten ungetesteten Punkte. Nutze dafür das Issue-Template „Kompatibilitätsmeldung“, sobald es verfügbar ist, oder ein normales GitHub-Issue.
