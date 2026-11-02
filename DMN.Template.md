# d.velop Process Studio - DMN Template

## Übersicht
Dieses Template beschreibt Struktur und Konventionen für Entscheidungstabellen (DMN) in d.velop Process Studio. Grundlage ist der Export des d.velop process modeler ([example-DMN/Standort zu Prüfer_draft.dmn](example-DMN/Standort%20zu%20Pr%C3%BCfer_draft.dmn)).

Das öffentliche Administrationshandbuch (Version 2023.Q4) dokumentiert DMN nicht. Inhalte sind daher gekennzeichnet:

| Kennzeichnung | Bedeutung |
|---|---|
| **[V]** | Verifiziert durch Beispielexport oder Handbuch |
| **[U]** | Ungetestet – DMN-1.3/Camunda-7-Standard, Verhalten in d.velop Process Studio im Zielsystem prüfen |

## Inhaltsverzeichnis
1. [Grundstruktur](#grundstruktur)
2. [IDs und Namen](#ids-und-namen)
3. [Eingaben (Input)](#eingaben-input)
4. [Ausgaben (Output)](#ausgaben-output)
5. [Datentypen](#datentypen)
6. [Regeln und Ausdrücke](#regeln-und-ausdrücke)
7. [Identity-Werte](#identity-werte)
8. [Hit Policies](#hit-policies)
9. [Mehrere Eingaben und Ausgaben](#mehrere-eingaben-und-ausgaben)
10. [Diagramm (DMNDI)](#diagramm-dmndi)
11. [Einbindung in BPMN](#einbindung-in-bpmn)
12. [Namenskonventionen](#namenskonventionen)
13. [Vollständiges Beispiel](#vollständiges-beispiel)
14. [Häufige Fehler](#häufige-fehler)
15. [Prüfliste](#prüfliste)

---

## Grundstruktur
**[V]** Platzhalter sind mit `{{…}}` gekennzeichnet.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<dmn:definitions xmlns:dmn="https://www.omg.org/spec/DMN/20191111/MODEL/" xmlns:camunda="http://camunda.org/schema/1.0/dmn" xmlns:dmndi="https://www.omg.org/spec/DMN/20191111/DMNDI/" xmlns:dc="http://www.omg.org/spec/DMN/20180521/DC/" id="b-{{UUID}}" name="{{TABELLENNAME}}" namespace="http://camunda.org/schema/1.0/dmn" exporter="d.velop process modeler" exporterVersion="1.1.0">
  <dmn:decision id="b-{{UUID}}-root" name="Main">
    <dmn:decisionTable id="mainTable">
      <!-- Eingaben, Ausgaben, Regeln -->
    </dmn:decisionTable>
  </dmn:decision>
  <dmndi:DMNDI>
    <dmndi:DMNDiagram>
      <dmndi:DMNShape dmnElementRef="b-{{UUID}}-root">
        <dc:Bounds height="80" width="180" x="160" y="120" />
      </dmndi:DMNShape>
    </dmndi:DMNDiagram>
  </dmndi:DMNDI>
</dmn:definitions>
```

Feste Werte **[V]**:
- Präfix `dmn:` für alle Modellelemente (DMN 1.3).
- `namespace="http://camunda.org/schema/1.0/dmn"`
- `exporter="d.velop process modeler"`, `exporterVersion="1.1.0"`
- Genau eine `dmn:decision` mit `name="Main"` und genau eine `dmn:decisionTable` mit `id="mainTable"`.

---

## IDs und Namen
| Element | ID-Format | Beispiel |
|---|---|---|
| `dmn:definitions` | `b-<UUID v4>` | `b-37e28fd2-1292-4080-a74b-cc411e1477c7` |
| `dmn:decision` | `<definitions-id>-root` | `b-37e28fd2-…-cc411e1477c7-root` |
| `dmn:decisionTable` | `mainTable` | `mainTable` |
| `dmn:input` | `input` (bei mehreren: `input_1`, `input_2` … **[U]**) | `input` |
| `dmn:inputExpression` | `inputExpression` (bei mehreren: `inputExpression_1` … **[U]**) | `inputExpression` |
| `dmn:output` | `output` (bei mehreren: `output_1` … **[U]**) | `output` |
| `dmn:rule` | `DecisionRule_<7 Zeichen a-z0-9>` oder `DecisionRule_<UUID>` | `DecisionRule_07t0xo2` |
| `dmn:inputEntry` | `UnaryTests_<7 Zeichen a-z0-9>` | `UnaryTests_1gcnmnh` |
| `dmn:outputEntry` | `LiteralExpression_<7 Zeichen a-z0-9>` | `LiteralExpression_1eci7xw` |

- `name` von `dmn:definitions` ist der Anzeigename der Tabelle (Umlaute erlaubt, z.B. `Standort zu Prüfer`).
- Alle IDs müssen innerhalb der Datei eindeutig sein und mit einem Buchstaben beginnen.

---

## Eingaben (Input)
**[V]**
```xml
<dmn:input id="input" label="{{EINGABE_VARIABLE}}" camunda:inputVariable="input">
  <dmn:inputExpression id="inputExpression" typeRef="{{TYP}}">
    <dmn:text></dmn:text>
  </dmn:inputExpression>
</dmn:input>
```
- `label` = Name der Prozessvariable, deren Wert geprüft wird (z.B. `Standort_GR`).
- `camunda:inputVariable="input"` und leerer `<dmn:text></dmn:text>` in der `inputExpression` wie im Export beibehalten. Die Zuordnung zur Prozessvariable erfolgt über `label`.
- `typeRef` siehe [Datentypen](#datentypen).

---

## Ausgaben (Output)
**[V]**
```xml
<dmn:output id="output" label="{{AUSGABE_VARIABLE}}" name="output" typeRef="{{TYP}}" />
```
- `label` = Name der Prozessvariable, die das Ergebnis erhält (z.B. `Pruefer_GR`).
- `name="output"` unverändert lassen.
- Die Ausgabevariable sollte im BPMN-Prozess deklariert sein (`variable:<name>` mit passendem Typ).

---

## Datentypen
Zulässige `typeRef`-Werte – **großgeschrieben** wie die d.velop-Prozessvariablentypen:

| typeRef | Status | Wertformat in Regeln | Beispiel |
|---|---|---|---|
| `String` | [V] | Text in doppelten Anführungszeichen | `"intern"` |
| `Identity` | [V] (Ausgabe) / [U] (Eingabe) | Identity-URI in Anführungszeichen | `"identity:///identityprovider/scim/Users/<id>"` |
| `Number` | [U] | Zahl ohne Anführungszeichen, Dezimalpunkt | `1500.50` |
| `Boolean` | [U] | `true` / `false` ohne Anführungszeichen | `true` |

- Keine Camunda-Typen wie `string`, `integer`, `double`, `boolean`, `date` verwenden.
- Es gibt keinen Datumstyp für Prozessvariablen (Handbuch 2023.Q4).

---

## Regeln und Ausdrücke
**[V]** Eine Regel enthält je Eingabe einen `inputEntry` und je Ausgabe einen `outputEntry` – in der Reihenfolge der Spalten.

```xml
<dmn:rule id="DecisionRule_{{ID}}">
  <dmn:inputEntry id="UnaryTests_{{ID}}">
    <dmn:text>{{BEDINGUNG}}</dmn:text>
  </dmn:inputEntry>
  <dmn:outputEntry id="LiteralExpression_{{ID}}">
    <dmn:text>{{ERGEBNIS}}</dmn:text>
  </dmn:outputEntry>
</dmn:rule>
```

### Bedingungen (inputEntry)
| Ausdruck | Bedeutung | Status |
|---|---|---|
| `"intern"` | Wert ist exakt `intern` | [V] |
| *(leer)* `<dmn:text></dmn:text>` | beliebiger Wert (Auffangregel) | [V] |
| `"a","b"` | Wert ist `a` oder `b` | [U] |
| `not("a")` | Wert ist nicht `a` | [U] |
| `< 1000`, `>= 1000` | Zahlenvergleich | [U] |
| `[1000..5000]` | Bereich inklusive Grenzen | [U] |
| `]1000..5000[` | Bereich exklusive Grenzen | [U] |
| `true` / `false` | Boolescher Wert | [U] |

### Ergebnisse (outputEntry)
- Literal passend zum `typeRef` (siehe [Datentypen](#datentypen)).
- Kein leerer Ausgabewert.

### Regelreihenfolge
- Spezifische Regeln zuerst, Auffangregel (leere Bedingung) **zuletzt** **[V]**.
- Sonderzeichen in Texten XML-escapen: `&` → `&amp;`, `<` → `&lt;`, `>` → `&gt;`.

---

## Identity-Werte
**[V]** Benutzer und Gruppen werden als Identity-URI angegeben:

| Ziel | Format |
|---|---|
| Benutzer | `"identity:///identityprovider/scim/Users/<Benutzer-ID>"` |
| Gruppe | `"identity:///identityprovider/scim/Groups/<Gruppen-ID>"` |

- Schreibweise `Users`/`Groups` wie im Modeler-Export verwenden (das Handbuch zeigt in BPMN-Kontexten `users`/`groups`; Kleinschreibung in DMN **[U]**).
- IDs stammen aus der Benutzerverwaltung des Zielsystems. Unbekannte IDs als Platzhalter `{{BENUTZER_ID_<ROLLE>}}` bzw. `{{GRUPPEN_ID_<ROLLE>}}` kennzeichnen und vor dem Import ersetzen.

---

## Hit Policies
**[V]** Ohne Attribut `hitPolicy` gilt `UNIQUE` (genau eine Regel darf zutreffen). Der Export setzt kein Attribut.

**[U]** Weitere Policies am `dmn:decisionTable`-Element:

| hitPolicy | aggregation | Verhalten |
|---|---|---|
| *(keins)* / `UNIQUE` | – | Genau eine Regel trifft zu; Überschneidungen sind Fehler |
| `FIRST` | – | Erste zutreffende Regel in Reihenfolge |
| `ANY` | – | Mehrere Treffer erlaubt, wenn Ausgaben identisch |
| `PRIORITY` | – | Treffer nach Priorität der Ausgabewerte (`outputValues` nötig) |
| `COLLECT` | – / `SUM` / `MIN` / `MAX` / `COUNT` | Alle Treffer als Liste bzw. aggregiert |
| `RULE ORDER` | – | Alle Treffer als Liste in Regelreihenfolge |
| `OUTPUT ORDER` | – | Alle Treffer nach Priorität der Ausgabewerte |

```xml
<dmn:decisionTable id="mainTable" hitPolicy="FIRST">
<dmn:decisionTable id="mainTable" hitPolicy="COLLECT" aggregation="SUM">
```

Hinweis: Bei `UNIQUE` muss die Auffangregel (leere Bedingung) alle übrigen Fälle abdecken, ohne sich mit anderen Regeln zu überschneiden. Eine leere Bedingung überschneidet sich technisch mit jeder anderen Regel – im Beispielexport ist das dennoch so modelliert **[V]**. Bei Fehlern im Zielsystem `hitPolicy="FIRST"` verwenden **[U]**.

---

## Mehrere Eingaben und Ausgaben
**[U]** Mehrere Spalten sind nach DMN-Standard möglich; ob der d.velop-Editor sie unterstützt, ist ungeprüft.

```xml
<dmn:decisionTable id="mainTable" hitPolicy="FIRST">
  <dmn:input id="input_1" label="Standort" camunda:inputVariable="input">
    <dmn:inputExpression id="inputExpression_1" typeRef="String">
      <dmn:text></dmn:text>
    </dmn:inputExpression>
  </dmn:input>
  <dmn:input id="input_2" label="Betrag" camunda:inputVariable="input">
    <dmn:inputExpression id="inputExpression_2" typeRef="Number">
      <dmn:text></dmn:text>
    </dmn:inputExpression>
  </dmn:input>
  <dmn:output id="output_1" label="Pruefer" name="output_1" typeRef="Identity" />
  <dmn:output id="output_2" label="Stufe" name="output_2" typeRef="String" />
  <dmn:rule id="DecisionRule_a1b2c3d">
    <dmn:inputEntry id="UnaryTests_a1b2c3e"><dmn:text>"intern"</dmn:text></dmn:inputEntry>
    <dmn:inputEntry id="UnaryTests_a1b2c3f"><dmn:text>&gt;= 1000</dmn:text></dmn:inputEntry>
    <dmn:outputEntry id="LiteralExpression_a1b2c3g"><dmn:text>"identity:///identityprovider/scim/Users/{{BENUTZER_ID}}"</dmn:text></dmn:outputEntry>
    <dmn:outputEntry id="LiteralExpression_a1b2c3h"><dmn:text>"hoch"</dmn:text></dmn:outputEntry>
  </dmn:rule>
</dmn:decisionTable>
```
- Anzahl `inputEntry` je Regel = Anzahl `dmn:input`; Anzahl `outputEntry` = Anzahl `dmn:output`.
- Bei mehreren Ausgaben eindeutige `name`-Werte (`output_1`, `output_2`) verwenden.

---

## Diagramm (DMNDI)
**[V]** Genau ein `DMNShape` für die Entscheidung:
```xml
<dmndi:DMNShape dmnElementRef="b-{{UUID}}-root">
  <dc:Bounds height="80" width="180" x="160" y="120" />
</dmndi:DMNShape>
```
`dmnElementRef` muss exakt der Decision-ID entsprechen.

---

## Einbindung in BPMN
**Status: ausstehend.** Eine Referenz-BPMN mit Geschäftsregelaktivität (Export aus dem d.velop process modeler) wird in `example-DMN/` abgelegt. Bis dahin gilt:

- Die Einbindung ist **nicht** im Handbuch 2023.Q4 dokumentiert; `businessRuleTask` und `camunda:decisionRef` fehlen dort in der Liste der erlaubten Elemente/Erweiterungen.
- Liegt eine Referenz-BPMN vor, sind deren Elementtyp, Attribute und Layout exakt zu übernehmen.
- Ohne Referenz die Einbindung **nicht** erzeugen, sondern nur beschreiben (Decision-ID, Eingabe- und Ausgabevariablen).

**[U]** Camunda-7-Muster zur Orientierung:
```xml
<businessRuleTask id="task_{{OBJEKT}}_ermitteln" name="{{OBJEKT}} ermitteln"
                  camunda:decisionRef="b-{{UUID}}-root"
                  camunda:mapDecisionResult="singleEntry"
                  camunda:resultVariable="{{AUSGABE_VARIABLE}}" />
```
- `decisionRef` verweist auf die **Decision-ID** (`…-root`), nicht auf die Definitions-ID.

---

## Namenskonventionen
- Dateiablage: `CreateDMN/<Tabellenname>.dmn`, Leerzeichen → `_`, Umlaute → `ae`/`oe`/`ue`/`ss` (z.B. `Standort_zu_Pruefer.dmn`).
- Tabellenname (`name`): fachlich, Muster `<Eingabe> zu <Ergebnis>` (z.B. `Standort zu Prüfer`).
- Variablennamen in `label`: identisch zu den Prozessvariablen im BPMN, ohne Umlaute und Leerzeichen.

---

## Vollständiges Beispiel
**[V]** Zuordnung eines Prüfers anhand des Standorts:

| Standort_GR (String) | Pruefer_GR (Identity) |
|---|---|
| `"intern"` | Benutzer `e13fd05f-…` |
| `"extern"` | Benutzer `4d1d33cf-…` |
| *(beliebig)* | Gruppe `6DB690CB-…` |

```xml
<?xml version="1.0" encoding="UTF-8"?>
<dmn:definitions xmlns:dmn="https://www.omg.org/spec/DMN/20191111/MODEL/" xmlns:camunda="http://camunda.org/schema/1.0/dmn" xmlns:dmndi="https://www.omg.org/spec/DMN/20191111/DMNDI/" xmlns:dc="http://www.omg.org/spec/DMN/20180521/DC/" id="b-37e28fd2-1292-4080-a74b-cc411e1477c7" name="Standort zu Prüfer" namespace="http://camunda.org/schema/1.0/dmn" exporter="d.velop process modeler" exporterVersion="1.1.0">
  <dmn:decision id="b-37e28fd2-1292-4080-a74b-cc411e1477c7-root" name="Main">
    <dmn:decisionTable id="mainTable">
      <dmn:input id="input" label="Standort_GR" camunda:inputVariable="input">
        <dmn:inputExpression id="inputExpression" typeRef="String">
          <dmn:text></dmn:text>
        </dmn:inputExpression>
      </dmn:input>
      <dmn:output id="output" label="Pruefer_GR" name="output" typeRef="Identity" />
      <dmn:rule id="DecisionRule_07t0xo2">
        <dmn:inputEntry id="UnaryTests_1gcnmnh">
          <dmn:text>"intern"</dmn:text>
        </dmn:inputEntry>
        <dmn:outputEntry id="LiteralExpression_1eci7xw">
          <dmn:text>"identity:///identityprovider/scim/Users/e13fd05f-99fa-4516-9cb1-ee41c1d3dda7"</dmn:text>
        </dmn:outputEntry>
      </dmn:rule>
      <dmn:rule id="DecisionRule_fe6d503a-0de8-21c6-9562-57cf28ab7140">
        <dmn:inputEntry id="UnaryTests_0qjogj7">
          <dmn:text>"extern"</dmn:text>
        </dmn:inputEntry>
        <dmn:outputEntry id="LiteralExpression_091bwao">
          <dmn:text>"identity:///identityprovider/scim/Users/4d1d33cf-186d-4c40-a3cd-7143ec921e5f"</dmn:text>
        </dmn:outputEntry>
      </dmn:rule>
      <dmn:rule id="DecisionRule_c3203cba-a29e-186a-972c-a3c8db0cb3ac">
        <dmn:inputEntry id="UnaryTests_0qe9s17">
          <dmn:text></dmn:text>
        </dmn:inputEntry>
        <dmn:outputEntry id="LiteralExpression_09g1jjh">
          <dmn:text>"identity:///identityprovider/scim/Groups/6DB690CB-EA1B-4D45-B00B-63A2E7B21816"</dmn:text>
        </dmn:outputEntry>
      </dmn:rule>
    </dmn:decisionTable>
  </dmn:decision>
  <dmndi:DMNDI>
    <dmndi:DMNDiagram>
      <dmndi:DMNShape dmnElementRef="b-37e28fd2-1292-4080-a74b-cc411e1477c7-root">
        <dc:Bounds height="80" width="180" x="160" y="120" />
      </dmndi:DMNShape>
    </dmndi:DMNDiagram>
  </dmndi:DMNDI>
</dmn:definitions>
```

---

## Häufige Fehler
| Fehler | Korrektur |
|---|---|
| `typeRef="string"` (klein) | `typeRef="String"` |
| Strings ohne Anführungszeichen (`intern`) | `"intern"` |
| Zahlen/Booleans in Anführungszeichen (`"100"`) | `100`, `true` |
| Decision-ID ohne `-root` oder abweichend vom `DMNShape` | `b-<uuid>-root` an beiden Stellen |
| `label` weicht vom Variablennamen im BPMN ab | Exakt gleiche Schreibweise |
| Anzahl `inputEntry`/`outputEntry` passt nicht zu den Spalten | Pro Spalte genau ein Eintrag je Regel |
| Keine Auffangregel → kein Ergebnis bei unbekanntem Wert | Letzte Regel mit leerer Bedingung |
| Überschneidende Regeln bei `UNIQUE` | Bedingungen disjunkt formulieren oder `FIRST` **[U]** |
| `>`/`<`/`&` unescaped | `&gt;`, `&lt;`, `&amp;` |
| Erfundene Benutzer-/Gruppen-IDs | Platzhalter `{{BENUTZER_ID_…}}` verwenden |
| Template-Platzhalter `{{…}}` in IDs vergessen | Alle Platzhalter ersetzen |

---

## Prüfliste
- [ ] Wohlgeformtes XML mit XML-Deklaration und allen vier Namespaces
- [ ] `exporter="d.velop process modeler"`, `exporterVersion="1.1.0"`, `namespace` gesetzt
- [ ] Definitions-ID `b-<uuid>`, Decision-ID `b-<uuid>-root`, `DMNShape` verweist darauf
- [ ] Alle IDs eindeutig
- [ ] `typeRef` nur `String`, `Number`, `Boolean`, `Identity`
- [ ] Input-/Output-`label` entsprechen Prozessvariablen
- [ ] Je Regel genau ein `inputEntry` pro Eingabe und ein `outputEntry` pro Ausgabe
- [ ] Literale passend zum Typ; Identity-URIs vollständig
- [ ] Auffangregel am Ende (sofern fachlich gewünscht)
- [ ] Keine Überschneidungen bei `UNIQUE`; ungetestete Features als Risiko gemeldet
- [ ] Keine verbleibenden `{{…}}`-Platzhalter außer bewusst markierten Benutzer-/Gruppen-IDs
