# d.velop Process Studio – Skript-Template (d.velop scripting)

## Übersicht
Dieses Template beschreibt Struktur und Konventionen für Skripte in d.velop scripting (Bestandteil von d.velop process studio). Grundlage sind das Administrationshandbuch **2025.Q1**, Kapitel „Verwenden von Skripten“, die d.velop-Hilfeartikel, das Developer-Portal (siehe [Quellen](#quellen)) sowie das Export-Beispiel `examples/example-script/SendNotification_draft.json`.

Erzeugt werden **Skript-Exportdateien (`.json`)**, die über die Importschaltfläche der Skriptverwaltung eingelesen werden. Der JavaScript-Code steht im Feld `content` (siehe [Exportformat (JSON)](#exportformat-json)). Das Template gilt **nur für die Cloud**.

| Kennzeichnung | Bedeutung |
|---|---|
| **[V]** | Verifiziert durch Handbuch, Hilfeartikel, Referenzcode der d.velop-Dokumentation oder das Export-Beispiel |
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
12. [Exportformat (JSON)](#exportformat-json)
13. [Namenskonventionen](#namenskonventionen)
14. [Vollständiges Beispiel](#vollständiges-beispiel)
15. [Häufige Fehler](#häufige-fehler)
16. [Prüfliste](#prüfliste)
17. [Quellen](#quellen)

---

## Laufzeitumgebung
- **[V]** Skripte laufen auf **Node.js**. In der Cloud wird seit dem 31.08.2025 ausschließlich **Node.js 22** unterstützt (zuvor Node 18).
- **[V]** Module werden mit `require(...)` geladen (CommonJS). Keine `import`-Syntax verwenden.
- **[V]** `async/await`, optional chaining `?.` und das globale `fetch` werden im Export-Beispiel verwendet.
- **[U]** Weitere Node-22-Sprachmittel (z.B. nullish coalescing `??`) sind verfügbar.
- On-Premises wird nicht unterstützt. Das Export-Beispiel enthält einen On-Prem-Fallback (`req.var('baseUri')`), der hier bewusst nicht übernommen wird.

---

## Grundgerüst
**[V]** Beim Anlegen eines Skripts erzeugt Process Studio ein lauffähiges „Hallo Welt“-Skript, das den Request-Body ausliest und „Hello World!“ zurückgibt.

**[V]** Funktionsrahmen laut Export-Beispiel:

```js
module.exports = async (req, res) => {
  const body = req.json();
  const name = body?.input?.yourName ?? "World";

  res.status(200).json({ output: { greeting: `Hello ${name}!` } });
};
```

- Die Funktion erhält das **Request-Objekt** `req` und das **Response-Objekt** `res`.
- **[V]** Hilfsfunktionen dürfen nach `module.exports` in derselben Datei stehen.
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
- **[V]** Das Export-Beispiel setzt nur `res.status(200)` ohne Body. Für Aufrufe ohne Rückgabewerte genügt das; empfohlen bleibt eine explizite Antwort.

---

## Variablen
**[V]**
- Variablen werden im Skript unter **Variablen → Variable hinzufügen** angelegt (Name, Wert, optional **verschlüsselt**).
- Zugriff im Code ausschließlich über `req.var("<Name>")` bzw. `req.variables()`.
- API-Schlüssel, Kennwörter und Tokens **immer** als verschlüsselte Variable ablegen, nie im Code.
- Beim Export werden verschlüsselte Variablen mit **leerem Wert** exportiert und müssen nach dem Import nachgepflegt werden.
- Im Export stehen Variablen unter `customerVariables` als `{ "key", "value", "encrypted" }`. Unverschlüsselte Variablen werden mit Wert exportiert.

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
- Für HTTP-Aufrufe das eingebaute `fetch` verwenden **[V]**.
- Im Export stehen Bibliotheken im Feld `dependencies` (ohne Bibliotheken: `null`) **[V]**. Das Format bei vorhandenen Bibliotheken ist nicht belegt **[U]** – generierte Exportdateien enthalten daher keine Bibliotheken. Werden Bibliotheken zwingend benötigt, nach dem Import manuell im Tab „Externe Bibliotheken“ ergänzen.

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
- **[V]** d.velop-APIs wie die Benachrichtigung erwarten die reine Benutzer-ID. Das Export-Beispiel schneidet dazu mit `substring(40)` ab; stattdessen den Präfix prüfen:

```js
const USER_PREFIX = "identity:///identityprovider/scim/users/";

function userIdFromIdentity(uri) {
  return typeof uri === "string" && uri.startsWith(USER_PREFIX) ? uri.slice(USER_PREFIX.length) : null;
}
```

- **[U]** Gruppen kommen als `identity:///identityprovider/scim/groups/<id>` und werden von `userIdFromIdentity` abgelehnt; bei Bedarf gesondert behandeln.
- **[V]** Das Export-Beispiel gibt keinen `output` zurück; für Service-Aufgaben ohne Rückgabewerte ist das ausreichend.
- Die Einbindung in BPMN (Service-Aufgabe, Skript-URL, Ein-/Ausgabezuordnung) erfolgt im d.velop process modeler und wird hier nur beschrieben, nicht generiert. Muster für Service-Aufrufe: `BPMN_Template.xml` (Abschnitt SERVICE TASK) und `examples/example-bpmn-processes/Sync Service`.

---

## Aufruf von d.velop-APIs
- **[V]** Basisadresse aus `x-dv-baseuri` lesen, nie fest eintragen.
- **[V]** **Standard: Anmeldung des Aufrufers weiterreichen.** Ist das Cookie `AuthSessionId` gesetzt, wird es als Bearer-Token verwendet, sonst der Header `Authorization` des Aufrufers. Aktionen erfolgen im Namen des Aufrufers.
- **[V]** Header `Origin` mit der Basisadresse mitsenden.
- **[U]** Bei Aufruf aus einem Prozess ist der Aufrufer die Prozess-Engine bzw. deren Benutzerkontext; Berechtigungen im Zielsystem prüfen.
- **Alternative: API-Key** aus dem d.ecs identity provider (`<baseuri>/identityprovider/config/apikey`) als Bearer-Token **[V]**, nur wenn kein Benutzerkontext vorliegt oder andere Rechte nötig sind. API-Key dann als **verschlüsselte Variable** hinterlegen.
- `fetch` wirft bei HTTP-Fehlern keine Ausnahme – `response.ok` prüfen.
- Authorization-Header, Cookies und Tokens nie loggen.
- Konkrete Endpunkte immer aus dem Developer-Portal (https://help.d-velop.de/dev/documentation) oder einem Export-Beispiel übernehmen; fehlende Pfade als Platzhalter `{{API_PFAD}}` kennzeichnen.

| Endpunkt | Methode | Content-Type | Body | Quelle |
|---|---|---|---|---|
| `/notification/notificationmessage` | `POST` | `application/hal+json` | `{ subject, message, messageHTML, recipients: ["<Benutzer-ID>"] }` | **[V]** Export-Beispiel |

```js
module.exports = async (req, res) => {
  const baseUri = req.get("x-dv-baseuri");

  const response = await fetch(`${baseUri}{{API_PFAD}}`, {
    headers: {
      Accept: "application/json",
      Origin: baseUri,
      Authorization: getAuthorizationHeader(req),
    },
  });
  if (!response.ok) {
    res.status(502).json({ error: "Abruf fehlgeschlagen." });
    return;
  }

  res.status(200).json({ output: { daten: await response.json() } });
};

function getAuthorizationHeader(req) {
  const sessionId = parseCookies(req).AuthSessionId;
  return sessionId ? `Bearer ${sessionId}` : req.get("authorization");
}

function parseCookies(req) {
  const list = {};
  const cookieHeader = req.get("cookie");
  if (!cookieHeader) return list;

  for (const cookie of cookieHeader.split(";")) {
    const [rawName, ...rest] = cookie.split("=");
    const name = rawName?.trim();
    const value = rest.join("=").trim();
    if (name && value) list[name] = decodeURIComponent(value);
  }
  return list;
}
```

---

## Fehlerbehandlung und Statuscodes
- Gesamte Logik in `try/catch` fassen; unbehandelte Ausnahmen vermeiden.
- Eingaben am Anfang validieren und mit `400` abbrechen.
- Fehlende Konfiguration (Variablen) mit `500` und eindeutiger Meldung melden.
- Fehler externer Systeme mit `502` weitergeben; keine Secrets, Tokens oder vollständigen Fremdantworten in Fehlermeldungen oder Logs ausgeben.
- `console.log`/`console.error` für Diagnose nutzen; keine personenbezogenen Daten loggen.
- Das Export-Beispiel wirft bei Fehlern eine Ausnahme (`throw new Error(...)`). Generierte Skripte setzen stattdessen explizite Statuscodes.

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
| `{METHOD} /script/{scriptId}/version/{versionId}/run` | Bestimmte Skriptversion ausführen **[V]** (Export-Beispiel, `action.endpoint`) |
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
- Das Exportformat ist im nächsten Abschnitt beschrieben. **[U]** Nicht freigegebene Skripte werden als `<Name>_draft.json` exportiert (abgeleitet aus dem Dateinamen des Beispiels).

---

## Exportformat (JSON)
**[V]** Felder laut `examples/example-script/SendNotification_draft.json`. Die Erzeugungsregeln gelten für neue Skripte im Entwurfsstatus.

| Feld | Typ | Erzeugungsregel |
|---|---|---|
| `scriptId` | UUID | Neue UUID v4 |
| `releaseId` | string | `""` (Entwurf) |
| `name` | string | Skriptname |
| `author`, `editor` | UUID | `""` **[U]** – Benutzer-IDs nie erfinden; Verhalten beim Import prüfen |
| `creationDate`, `changeDate` | string | Aktueller Zeitpunkt UTC, `YYYY-MM-DDTHH:MM:SSZ` |
| `releasedBy` | string | `""` |
| `releasedAt` | string | `"0001-01-01T00:00:00Z"` |
| `automationIds` | array | `[]` |
| `content` | string | Vollständiger JavaScript-Code als JSON-String, Zeilenumbruch `\r\n` |
| `dependencies` | null | `null` (siehe [Externe Bibliotheken](#externe-bibliotheken)) |
| `customerVariables` | array | `[{ "key": "<NAME>", "value": "<Wert oder leer>", "encrypted": <bool> }]`; verschlüsselte Variablen mit `""` |
| `action` | object | Siehe unten |
| `actionEnabled` | boolean | `false` |

`action` (Skript als d.velop Action; bleibt deaktiviert):

| Feld | Erzeugungsregel |
|---|---|
| `caller_type`, `input_properties`, `output_properties`, `tags` | `null` |
| `description`, `display_name` | `{ "de": "" }` |
| `endpoint` | `/scripting/script/<scriptId>/version/<versionId>/run` |
| `execution_mode` | `"Synchron"` |
| `id` | `<scriptId>-<versionId>` |
| `volatile` | `true` |

- `<versionId>` ist eine weitere neue UUID v4 und muss in `endpoint` und `id` identisch sein.
- **[U]** Ob der Import `scriptId`, `versionId`, `author` und `editor` übernimmt oder neu vergibt, ist ungeprüft.
- Feldreihenfolge wie im Beispiel beibehalten.

Grundgerüst:

```json
{
  "scriptId": "<scriptId>",
  "releaseId": "",
  "name": "<Skriptname>",
  "author": "",
  "editor": "",
  "creationDate": "<YYYY-MM-DDTHH:MM:SSZ>",
  "changeDate": "<YYYY-MM-DDTHH:MM:SSZ>",
  "releasedBy": "",
  "releasedAt": "0001-01-01T00:00:00Z",
  "automationIds": [],
  "content": "module.exports = async (req, res) => {\r\n    res.status(200).json({ output: {} });\r\n};\r\n",
  "dependencies": null,
  "customerVariables": [],
  "action": {
    "caller_type": null,
    "description": { "de": "" },
    "display_name": { "de": "" },
    "endpoint": "/scripting/script/<scriptId>/version/<versionId>/run",
    "execution_mode": "Synchron",
    "id": "<scriptId>-<versionId>",
    "input_properties": null,
    "output_properties": null,
    "tags": null,
    "volatile": true
  },
  "actionEnabled": false
}
```

Prüfung (JSON gültig, Felder wie im Beispiel, Code kompilierbar ohne Ausführung):

```powershell
node -e "const a=require('./examples/example-script/SendNotification_draft.json'),s=require('./CreateScript/<Datei>.json');const k=o=>Object.keys(o).sort().join();if(k(a)!==k(s)||k(a.action)!==k(s.action))throw new Error('Feldabweichung');new (require('vm').Script)(s.content);console.log('ok')"
```

---

## Namenskonventionen
- Dateiname: `CreateScript/<Skriptname>.json`; Leerzeichen → `_`, Umlaute → `ae`/`oe`/`ue`/`ss`.
- Feld `name`: Skriptname wie in der Skriptverwaltung angezeigt (Beispiel: `SendNotification`).
- Skriptvariablen: `UPPER_SNAKE_CASE` (z.B. `API_KEY`, `ZIEL_URL`).
- Schlüssel in `input`/`output`: identisch mit den Prozessvariablennamen im BPMN-Modell.
- Kopf im `content` mit Kurzbeschreibung, Variablen und Authentifizierung als Kommentar:

```js
// Skript: <Name> – <Zweck in einem Satz>
// Variablen: keine | API_KEY (verschlüsselt)
// Authentifizierung: Aufrufer | API-Key
```

---

## Vollständiges Beispiel
Benachrichtigung an einen Benutzer senden – überarbeitete Fassung von `examples/example-script/SendNotification_draft.json` (Aufrufer-Anmeldung, Eingabeprüfung, HTML-Escaping, explizite Statuscodes, ohne On-Prem-Fallback).

Code (Inhalt von `content`):

```js
// Skript: Benachrichtigung_senden – sendet eine d.velop-Benachrichtigung an einen Benutzer
// Variablen: keine
// Authentifizierung: Aufrufer
const USER_PREFIX = "identity:///identityprovider/scim/users/";

module.exports = async (req, res) => {
  let body;
  try {
    body = req.text() ? JSON.parse(req.text()) : {};
  } catch {
    res.status(400).json({ error: "Request-Body ist kein gültiges JSON." });
    return;
  }

  const { subject, message, recipient } = body.input ?? {};
  const recipientId = userIdFromIdentity(recipient);
  if (!subject || !message || !recipientId) {
    res.status(400).json({ error: "input.subject, input.message und input.recipient (Benutzer-Identity) sind erforderlich." });
    return;
  }

  const baseUri = req.get("x-dv-baseuri");
  if (!baseUri) {
    res.status(500).json({ error: "Header x-dv-baseuri fehlt." });
    return;
  }

  try {
    const response = await fetch(`${baseUri}/notification/notificationmessage`, {
      method: "POST",
      headers: {
        "Content-Type": "application/hal+json",
        Origin: baseUri,
        Authorization: getAuthorizationHeader(req),
      },
      body: JSON.stringify({
        subject,
        message,
        messageHTML: escapeHtml(message),
        recipients: [recipientId],
      }),
    });
    if (!response.ok) {
      console.error("Benachrichtigung fehlgeschlagen, Status:", response.status);
      res.status(502).json({ error: "Benachrichtigung konnte nicht gesendet werden." });
      return;
    }
  } catch (err) {
    console.error("Benachrichtigung fehlgeschlagen:", err.message);
    res.status(502).json({ error: "Benachrichtigung konnte nicht gesendet werden." });
    return;
  }

  res.status(200).json({ output: { gesendet: true } });
};

function userIdFromIdentity(uri) {
  return typeof uri === "string" && uri.startsWith(USER_PREFIX) ? uri.slice(USER_PREFIX.length) : null;
}

function escapeHtml(text) {
  const map = { "&": "&amp;", "<": "&lt;", ">": "&gt;", '"': "&quot;", "'": "&#39;" };
  return String(text).replace(/[&<>"']/g, (c) => map[c]);
}

function getAuthorizationHeader(req) {
  const sessionId = parseCookies(req).AuthSessionId;
  return sessionId ? `Bearer ${sessionId}` : req.get("authorization");
}

function parseCookies(req) {
  const list = {};
  const cookieHeader = req.get("cookie");
  if (!cookieHeader) return list;

  for (const cookie of cookieHeader.split(";")) {
    const [rawName, ...rest] = cookie.split("=");
    const name = rawName?.trim();
    const value = rest.join("=").trim();
    if (name && value) list[name] = decodeURIComponent(value);
  }
  return list;
}
```

Exportdatei `CreateScript/Benachrichtigung_senden.json`: Grundgerüst aus [Exportformat (JSON)](#exportformat-json) mit `"name": "Benachrichtigung_senden"`, neuen UUIDs, `"customerVariables": []` und dem obigen Code in `content`.

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
| Identity-URI mit festem Offset (`substring(40)`) gekürzt | Präfix prüfen (`userIdFromIdentity`) |
| Header `Origin` fehlt bei d.velop-API-Aufruf | `Origin: baseUri` mitsenden |
| `fetch`-Antwort nicht geprüft | `response.ok` auswerten |
| Eingaben ungeprüft in `messageHTML` | HTML escapen |
| `content` nicht korrekt als JSON-String escaped | Anführungszeichen, Backslashes und Zeilenumbrüche (`\r\n`) escapen; per Prüfbefehl kontrollieren |
| `versionId` in `action.endpoint` und `action.id` verschieden | Dieselbe UUID verwenden |
| Wert bei verschlüsselter Variable im Export | `"value": ""` |

---

## Prüfliste
- [ ] Datei `CreateScript/<Name>.json`, Name nach Konvention.
- [ ] Prüfbefehl aus [Exportformat (JSON)](#exportformat-json) ohne Fehler (JSON, Felder, Code).
- [ ] `scriptId`/`versionId` neu erzeugt; `action.endpoint` und `action.id` konsistent.
- [ ] Entwurfsstatus: `releaseId`/`releasedBy` leer, `releasedAt` = `0001-01-01T00:00:00Z`; `dependencies` = `null`.
- [ ] Nur `require`, keine `import`-Syntax; keine externen Bibliotheken.
- [ ] Body sicher geparst; Eingaben validiert (`400`).
- [ ] Anmeldung des Aufrufers weitergereicht (oder API-Key begründet), Header `Origin` gesetzt.
- [ ] Alle Secrets als verschlüsselte Variablen mit leerem Wert; keine Zugangsdaten im Code.
- [ ] Basisadresse über `x-dv-baseuri`.
- [ ] Jeder Pfad setzt einen Statuscode und sendet genau eine Antwort.
- [ ] Fehler externer Aufrufe abgefangen; keine Secrets/personenbezogenen Daten in Logs.
- [ ] `input`-/`output`-Schlüssel entsprechen den Prozessvariablen.
- [ ] Variablen und Authentifizierung im Kopfkommentar von `content`.
- [ ] Platzhalter `{{…}}` vollständig benannt.
- [ ] Importiert und im Testbereich mit Test-Payload ausgeführt.

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
- Export-Beispiel aus Process Studio: `examples/example-script/SendNotification_draft.json` (Exportformat, Funktionsrahmen, Aufrufer-Anmeldung, Benachrichtigungs-API)
