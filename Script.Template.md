# d.velop Process Studio – Skript-Template (d.velop scripting)

## Übersicht
Dieses Template beschreibt Struktur und Konventionen für Skripte in d.velop scripting (Bestandteil von d.velop process studio). Grundlage ist das Administrationshandbuch **2025.Q1**, Kapitel „Verwenden von Skripten“, sowie die d.velop-Hilfeartikel und das Developer-Portal (siehe [Quellen](#quellen)).

Ein Export-Beispiel (Skript-JSON) liegt im Repository nicht vor. Erzeugt werden daher nur **JavaScript-Dateien (`.js`)**, deren Inhalt in den Skripteditor kopiert wird. Variablen und externe Bibliotheken werden im Editor manuell angelegt.

| Kennzeichnung | Bedeutung |
|---|---|
| **[V]** | Verifiziert durch Handbuch, Hilfeartikel oder Referenzcode der d.velop-Dokumentation |
| **[U]** | Ungetestet – plausibel abgeleitet, Verhalten im Zielsystem prüfen |

## Inhaltsverzeichnis
1. [Laufzeitumgebung](#laufzeitumgebung)
2. [Grundgerüst](#grundgerüst)
3. [Request-Objekt](#request-objekt)
4. [Response-Objekt](#response-objekt)
5. [Variablen](#variablen)
6. [Externe Bibliotheken](#externe-bibliotheken)
7. [Aufruf aus einem Prozess](#aufruf-aus-einem-prozess)
8. [Aufruf von d.velop-APIs](#aufruf-von-dvelop-apis)
9. [Fehlerbehandlung und Statuscodes](#fehlerbehandlung-und-statuscodes)
10. [Aufruf, Logs und API-Endpunkte](#aufruf-logs-und-api-endpunkte)
11. [Verwaltung, Export und Import](#verwaltung-export-und-import)
12. [Namenskonventionen](#namenskonventionen)
13. [Vollständiges Beispiel](#vollständiges-beispiel)
14. [Häufige Fehler](#häufige-fehler)
15. [Prüfliste](#prüfliste)
16. [Quellen](#quellen)

---

## Laufzeitumgebung
- **[V]** Skripte laufen auf **Node.js**. In der Cloud wird seit dem 31.08.2025 ausschließlich **Node.js 22** unterstützt (zuvor Node 18).
- **[V]** Module werden mit `require(...)` geladen (CommonJS). Keine `import`-Syntax verwenden.
- **[U]** Node-22-Sprachmittel (`async/await`, optional chaining `?.`, nullish coalescing `??`, globales `fetch`) sind verfügbar.
- On-Premises: Node-Version der eigenen Installation prüfen (Offline-Modus siehe Handbuch, Abschnitt „Response-Objekt“). Nicht Gegenstand dieses Templates.

---

## Grundgerüst
**[V]** Beim Anlegen eines Skripts erzeugt Process Studio ein lauffähiges „Hallo Welt“-Skript, das den Request-Body ausliest und „Hello World!“ zurückgibt.

**[U]** Der exakte Funktionsrahmen ist öffentlich nicht dokumentiert. Vor der ersten Verwendung mit dem im eigenen System erzeugten „Hallo Welt“-Skript abgleichen und dieses Template ggf. korrigieren. Angenommene Form:

```js
module.exports = async (req, res) => {
  const body = req.json();
  const name = body?.input?.yourName ?? "World";

  res.status(200).json({ output: { greeting: `Hello ${name}!` } });
};
```

- Die Funktion erhält das **Request-Objekt** `req` und das **Response-Objekt** `res`.
- Asynchrone Aufrufe mit `await` abschließen, bevor die Funktion endet.

---

## Request-Objekt
**[V]** Referenz laut Handbuch (gekürzt):

| Mitglied | Rückgabe | Beschreibung |
|---|---|---|
| `req.method` | `string` | HTTP-Methode (`GET`, `POST`, `PUT`, `PATCH`, `DELETE`) |
| `req.path` | `string` | Aufgerufener Pfad |
| `req.params` | `Object` | Alle Anfrageparameter |
| `req.param(name)` | `string` | Anfrageparameter; Name wird **kleingeschrieben** nachgeschlagen; ohne Argument alle Parameter |
| `req.get(name)` | `string` | Header-Wert; Name wird **kleingeschrieben** nachgeschlagen; ohne Argument alle Header |
| `req.var(name)` | `string` | Skriptvariable (Groß-/Kleinschreibung beachten) |
| `req.variables()` | `Object` | Alle Skriptvariablen |
| `req.data()` | `Buffer` | Roher Request-Body |
| `req.text()` | `string` | Body als UTF-8-Text |
| `req.json()` | `Object` | Body per `JSON.parse` – **wirft bei leerem oder ungültigem Body** |
| `req.end()` | – | Veraltet, nicht verwenden |

- **[V]** Bei `GET` und `DELETE` kann kein Request-Body übergeben werden – Daten dann über Anfrageparameter.
- Sicheres Parsen:

```js
function readJson(req) {
  const text = req.text();
  if (!text) return {};
  try {
    return JSON.parse(text);
  } catch {
    return null;
  }
}
```

---

## Response-Objekt
**[V]** Referenz laut Handbuch (gekürzt):

| Methode | Beschreibung |
|---|---|
| `res.status(code)` | HTTP-Statuscode setzen; verkettbar |
| `res.json(obj)` | Setzt `Content-Type: application/json` und sendet `obj` |
| `res.send(body)` | Sendet `string`, `Buffer` oder Objekt (Objekt wird als JSON serialisiert) |
| `res.set(key, value)` | Header setzen; verkettbar. `res.set({ ... })` ersetzt alle Header |
| `res.end()` | Veraltet, nicht verwenden |

```js
res.status(200).json({ output: { ergebnis: "ok" } });
res.status(200).set("Content-Type", "text/plain; charset=utf-8").send("Hallo");
```

- Immer explizit einen Statuscode setzen (der interne Startwert ist `0`).
- `res.json()` gibt nichts zurück – nicht weiter verketten.

---

## Variablen
**[V]**
- Variablen werden im Skript unter **Variablen → Variable hinzufügen** angelegt (Name, Wert, optional **verschlüsselt**).
- Zugriff im Code ausschließlich über `req.var("<Name>")` bzw. `req.variables()`.
- API-Schlüssel, Kennwörter und Tokens **immer** als verschlüsselte Variable ablegen, nie im Code.
- Beim Export werden verschlüsselte Variablen mit **leerem Wert** exportiert und müssen nach dem Import nachgepflegt werden.

```js
const apiKey = req.var("API_KEY");
if (!apiKey) {
  res.status(500).json({ error: "Variable API_KEY ist nicht gepflegt." });
  return;
}
```

---

## Externe Bibliotheken
**[V]**
1. Registerkarte **Externe Bibliotheken** → Plussymbol.
2. Paketnamen von [npmjs.com](https://www.npmjs.com/) angeben, optional mit Version. Ohne Version wird die neueste übernommen.
3. Im Code per `require("<paket>")` laden.

- d.velop scripting muss `npmjs.com` erreichen können.
- Version immer explizit festlegen, um unbeabsichtigte Updates zu vermeiden.
- Nur Bibliotheken einsetzen, die tatsächlich benötigt werden; für einfache HTTP-Aufrufe genügt das in Node 22 eingebaute `fetch` **[U]**. Das offizielle Beispiel verwendet `axios` **[V]**.

---

## Aufruf aus einem Prozess
- **[V]** Das offizielle Beispielskript nutzt einen Request-Body, der „die direkte Verwendung im Prozess ermöglicht“. Eingaben stehen unter `input`:

```json
{
  "input": {
    "subject": "Hallo",
    "message": "Text",
    "recipient": "identity:///identityprovider/scim/users/<IDP-ID>"
  }
}
```

- **[V]** Die Basisadresse des Systems steht im Header `x-dv-baseuri` (`req.get("x-dv-baseuri")`).
- **[U]** Rückgabe an den Prozess über `output`; die Schlüssel werden in der Service-Aufgabe Prozessvariablen zugeordnet:

```json
{ "output": { "<variable>": "<wert>" } }
```

- **[U]** Prozessvariablen vom Typ Identity werden als Identity-URI (`identity:///identityprovider/scim/users/<id>`) übergeben.
- Die Einbindung in BPMN (Service-Aufgabe, Skript-URL, Ein-/Ausgabezuordnung) erfolgt im d.velop process modeler und wird hier nur beschrieben, nicht generiert. Muster für Service-Aufrufe: `BPMN_Template.xml` (Abschnitt SERVICE TASK) und `example-bpmn-processes/Sync Service`.

---

## Aufruf von d.velop-APIs
- **[V]** Basisadresse aus `x-dv-baseuri` lesen, nie fest eintragen.
- **[V]** Authentifizierung mit API-Key aus dem d.ecs identity provider (`<baseuri>/identityprovider/config/apikey`) als Bearer-Token. Aktionen erfolgen im Namen des Benutzers, dem der API-Key gehört.
- API-Key als **verschlüsselte Variable** hinterlegen.
- Konkrete Endpunkte immer aus dem Developer-Portal (https://help.d-velop.de/dev/documentation) übernehmen; fehlende Pfade als Platzhalter `{{API_PFAD}}` kennzeichnen.

```js
const axios = require("axios");

module.exports = async (req, res) => {
  const baseUri = req.get("x-dv-baseuri");
  const apiKey = req.var("API_KEY");

  const response = await axios.get(`${baseUri}{{API_PFAD}}`, {
    headers: {
      Authorization: `Bearer ${apiKey}`,
      Accept: "application/json",
    },
  });

  res.status(200).json({ output: { daten: response.data } });
};
```

---

## Fehlerbehandlung und Statuscodes
- Gesamte Logik in `try/catch` fassen; unbehandelte Ausnahmen vermeiden.
- Eingaben am Anfang validieren und mit `400` abbrechen.
- Fehlende Konfiguration (Variablen) mit `500` und eindeutiger Meldung melden.
- Fehler externer Systeme mit `502` weitergeben; keine Secrets, Tokens oder vollständigen Fremdantworten in Fehlermeldungen oder Logs ausgeben.
- `console.log`/`console.error` für Diagnose nutzen; keine personenbezogenen Daten loggen.

| Code | Verwendung |
|---|---|
| `200` | Erfolg mit Antwort |
| `204` | Erfolg ohne Antwort-Body |
| `400` | Ungültige oder fehlende Eingaben |
| `404` | Angefragtes Objekt nicht gefunden |
| `500` | Konfigurations- oder interner Fehler |
| `502` | Fehler eines aufgerufenen Fremdsystems |

---

## Aufruf, Logs und API-Endpunkte
- **[V]** Jedes Skript hat eine Ausführungs-URL (Skriptliste → Dreipunktemenü → „URL in die Zwischenablage kopieren“).
- **[V]** Aufruf per `GET`, `POST`, `PUT`, `PATCH`, `DELETE`; nur Benutzer des Systems dürfen Skripte ausführen.
- **[V]** Scripting-API (`https://<tenant>.d-velop.cloud/scripting`, Bearer-Authentifizierung):

| Endpunkt | Zweck |
|---|---|
| `{METHOD} /script/{scriptId}/run` | Skript synchron ausführen |
| `{METHOD} /script/{scriptId}/background` | Skript im Hintergrund ausführen **[U: Verhalten]** |
| `GET /log/script/{scriptId}` | Logs eines Skripts |
| `GET /log/script/{scriptId}/execution/{executionId}` | Log einer Ausführung |
| `GET /script/{scriptId}/sbom` | Software-Stückliste (Bibliotheken) |

---

## Verwaltung, Export und Import
**[V]**
- Skriptverwaltung: Konfiguration → Aufgaben und Prozesse → Skripting → Skriptverwaltung (Rolle Administration) bzw. Process Studio → Tools → Skripte.
- Testen: Skript öffnen → **Testen** → **Durchführen**; Response und Statuscode werden angezeigt.
- Export über „Skript herunterladen“ (JSON-Datei), Import über die Importschaltfläche. Beim Import kann gewählt werden, ob Variablen mit importiert werden; vorhandene Variablen werden dann überschrieben.
- Das JSON-Exportformat ist nicht dokumentiert; Exportdateien werden in diesem Repository **nicht** erzeugt.

---

## Namenskonventionen
- Dateiname: `CreateScript/<Skriptname>.js`; Leerzeichen → `_`, Umlaute → `ae`/`oe`/`ue`/`ss`.
- Skriptvariablen: `UPPER_SNAKE_CASE` (z.B. `API_KEY`, `ZIEL_URL`).
- Schlüssel in `input`/`output`: identisch mit den Prozessvariablennamen im BPMN-Modell.
- Dateikopf mit Kurzbeschreibung, erwarteten Variablen und Bibliotheken als Kommentar:

```js
// Skript: <Name> – <Zweck in einem Satz>
// Variablen: API_KEY (verschlüsselt)
// Bibliotheken: axios@<version>
```

---

## Vollständiges Beispiel
Benachrichtigung über einen d.velop-API-Aufruf senden (Struktur nach offiziellem Hilfeartikel; Endpunkt als Platzhalter).

```js
// Skript: Benachrichtigung_senden – sendet eine Benachrichtigung an einen Benutzer
// Variablen: API_KEY (verschlüsselt)
// Bibliotheken: axios@<version>
const axios = require("axios");

module.exports = async (req, res) => {
  let body;
  try {
    body = req.text() ? JSON.parse(req.text()) : {};
  } catch {
    res.status(400).json({ error: "Request-Body ist kein gültiges JSON." });
    return;
  }

  const { subject, message, recipient } = body.input ?? {};
  if (!subject || !message || !recipient) {
    res.status(400).json({ error: "input.subject, input.message und input.recipient sind erforderlich." });
    return;
  }

  const apiKey = req.var("API_KEY");
  if (!apiKey) {
    res.status(500).json({ error: "Variable API_KEY ist nicht gepflegt." });
    return;
  }

  const baseUri = req.get("x-dv-baseuri");

  try {
    await axios.post(
      `${baseUri}{{API_PFAD_BENACHRICHTIGUNG}}`,
      { subject, message, recipients: [recipient] },
      { headers: { Authorization: `Bearer ${apiKey}`, "Content-Type": "application/json" } }
    );
  } catch (err) {
    console.error("Benachrichtigung fehlgeschlagen:", err.response?.status ?? err.message);
    res.status(502).json({ error: "Benachrichtigung konnte nicht gesendet werden." });
    return;
  }

  res.status(200).json({ output: { gesendet: true } });
};
```

Test-Payload für den Testbereich:

```json
{ "input": { "subject": "Hallo", "message": "Testnachricht", "recipient": "identity:///identityprovider/scim/users/<IDP-ID>" } }
```

---

## Häufige Fehler
| Fehler | Korrektur |
|---|---|
| `req.json()` bei leerem Body → Ausnahme | Body über `req.text()` prüfen und sicher parsen |
| Header mit Großbuchstaben erwartet | `req.get()` sucht kleingeschrieben – Header-Namen sind case-insensitive |
| Secrets im Code | Verschlüsselte Variable + `req.var()` |
| Basisadresse hart kodiert | `req.get("x-dv-baseuri")` |
| Kein Statuscode gesetzt | Immer `res.status(<code>)` |
| `res.end()` / `req.end()` | Veraltet, entfernen |
| `import`-Syntax | `require` verwenden |
| `await` vergessen | Asynchrone Aufrufe vor Funktionsende abschließen |
| Body bei `GET`/`DELETE` erwartet | Anfrageparameter (`req.param()`) verwenden |
| Bibliothek ohne Version | Version im Tab „Externe Bibliotheken“ festlegen |
| Nach Import leere Variablen | Verschlüsselte Variablen nachpflegen |

---

## Prüfliste
- [ ] Datei liegt in `CreateScript/`, Name nach Konvention.
- [ ] `node --check` ohne Fehler.
- [ ] Nur `require`, keine `import`-Syntax.
- [ ] Funktionsrahmen mit dem „Hallo Welt“-Skript des Zielsystems abgeglichen **[U]**.
- [ ] Body sicher geparst; Eingaben validiert (`400`).
- [ ] Alle Secrets als verschlüsselte Variablen; keine Zugangsdaten im Code.
- [ ] Basisadresse über `x-dv-baseuri`.
- [ ] Jeder Pfad setzt einen Statuscode und sendet genau eine Antwort.
- [ ] Fehler externer Aufrufe abgefangen; keine Secrets/personenbezogenen Daten in Logs.
- [ ] `input`-/`output`-Schlüssel entsprechen den Prozessvariablen.
- [ ] Liste benötigter Variablen und Bibliotheken (mit Version) im Dateikopf.
- [ ] Platzhalter `{{…}}` vollständig benannt.
- [ ] Im Testbereich mit Test-Payload ausgeführt.

---

## Quellen
Stand der Recherche: 11/2026.
- Administrationshandbuch 2025.Q1 – Verwenden von Skripten: https://help.d-velop.de/docs/de/pub/process-studio-admin/2025.q1/verwenden-von-skripten
  - Erste Schritte, Externe Bibliotheken, Variablen, Testen, Request-Objekt, Response-Objekt (Unterseiten)
- Hilfeartikel „How do I create a script?“: https://help.d-velop.de/process-studio/en/articles/clrqeebm555b20bvz8rjv1b6s
- Tipps und Tricks (u.a. eigene Certification Authority): https://help.d-velop.de/docs/de/pub/process-studio-admin/2023.q4/tipps-und-tricks
- Developer-Portal, d.velop scripting (OpenAPI): https://help.d-velop.de/dev/documentation/dvelop-scripting
- API-Übersicht: https://help.d-velop.de/dev/documentation
- Node.js-Version: https://help.d-velop.de/dev/news/dvelop-process-studio-nodejs-version
