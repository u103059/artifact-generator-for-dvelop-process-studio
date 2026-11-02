# d.velop Process Studio - Formular Template

## Übersicht
Dieses Template beschreibt die vollständige Struktur und Spezifikationen für die Erstellung von Formularen für d.velop Process Studio. Formulare werden im JSON-Format definiert und basieren auf dem Form.io Framework.

## Inhaltsverzeichnis
1. [Grundstruktur](#grundstruktur)
2. [Komponenten-Typen](#komponenten-typen)
3. [Layout-System](#layout-system)
4. [Validierungsregeln](#validierungsregeln)
5. [Bedingte Anzeige](#bedingte-anzeige)
6. [Namenskonventionen](#namenskonventionen)
7. [Vollständiges Beispiel](#vollständiges-beispiel)
8. [Häufige Fehler](#häufige-fehler)

---

## Grundstruktur

### JSON-Schema Hauptstruktur
```json
{
  "id": "UUID",
  "name": "Formularname",
  "author": "UUID",
  "lastEditor": "UUID",
  "creationDate": "ISO-8601 Timestamp",
  "lastModificationDate": "ISO-8601 Timestamp",
  "definition": "{JSON-String der Form-Definition}",
  "_links": {
    "edit": {"href": "/dforms/ui/forms/{id}/edit"},
    "self": {"href": "/dforms/api/forms/{id}"},
    "view": {"href": "/dforms/ui/forms/{id}/view"}
  }
}
```

### Form Definition (innerhalb des "definition" Strings)
```json
{
  "formioFormDefinition": {
    "display": "form",
    "components": []
  },
  "customCss": "",
  "dvfDefVersion": "1.0"
}
```

---

## Komponenten-Typen

d.velop Process Studio unterstützt folgende 16 Komponenten-Typen:

### 1. **TextField** (Einzeiliges Textfeld)
```json
{
  "type": "textfield",
  "key": "eindeutigerKey",
  "label": "Beschriftung",
  "placeholder": "Platzhalter Text",
  "input": true,
  "validate": {
    "required": false,
    "minLength": 0,
    "maxLength": 255,
    "pattern": ""
  }
}
```

**Verwendung:** Für kurze Texteingaben wie Namen, E-Mail-Adressen, etc.

### 2. **TextArea** (Mehrzeiliges Textfeld)
```json
{
  "type": "textarea",
  "key": "beschreibung",
  "label": "Beschreibung",
  "rows": 6,
  "input": true,
  "autoExpand": false,
  "wysiwyg": false,
  "validate": {
    "required": false,
    "minLength": "",
    "maxLength": "",
    "minWords": "",
    "maxWords": ""
  }
}
```

**Verwendung:** Für längere Textinhalte, Kommentare, Beschreibungen.

### 3. **Number** (Zahleneingabe)
```json
{
  "type": "number",
  "key": "betrag",
  "label": "Betrag",
  "input": true,
  "delimiter": false,
  "requireDecimal": false,
  "validate": {
    "required": false,
    "min": 0,
    "max": 999999,
    "step": "any"
  }
}
```

**Verwendung:** Für numerische Werte, Beträge, Mengen.

### 4. **Email** (E-Mail-Feld)
```json
{
  "type": "email",
  "key": "emailAdresse",
  "label": "E-Mail",
  "input": true,
  "kickbox": {
    "enabled": false
  },
  "validate": {
    "required": false
  }
}
```

**Verwendung:** Speziell für E-Mail-Adressen mit automatischer Validierung.

### 5. **PhoneNumber** (Telefonnummer)
```json
{
  "type": "phoneNumber",
  "key": "telefon",
  "label": "Telefonnummer",
  "input": true,
  "inputMask": "",
  "validate": {
    "required": false
  }
}
```

**Verwendung:** Für Telefonnummern mit optionaler Formatierung.

### 6. **Checkbox** (Kontrollkästchen)
```json
{
  "type": "checkbox",
  "key": "zustimmung",
  "label": "Ich stimme den Bedingungen zu",
  "input": true,
  "defaultValue": false,
  "validate": {
    "required": false
  }
}
```

**Verwendung:** Für Ja/Nein-Entscheidungen, Zustimmungen.

### 7. **Select** (Dropdown-Liste)
```json
{
  "type": "select",
  "key": "auswahl",
  "label": "Bitte wählen",
  "input": true,
  "data": {
    "values": [
      {"label": "Option 1", "value": "opt1"},
      {"label": "Option 2", "value": "opt2"},
      {"label": "Option 3", "value": "opt3"}
    ]
  },
  "dataSrc": "values",
  "template": "<span>{{ item.label }}</span>",
  "searchEnabled": true,
  "validate": {
    "required": false
  }
}
```

**Verwendung:** Für Auswahlmenüs mit vordefinierten Optionen.

### 8. **SelectBoxes** (Mehrfachauswahl)
```json
{
  "type": "selectboxes",
  "key": "mehrfachauswahl",
  "label": "Wählen Sie mehrere",
  "input": true,
  "values": [
    {"label": "Option A", "value": "optA"},
    {"label": "Option B", "value": "optB"},
    {"label": "Option C", "value": "optC"}
  ],
  "validate": {
    "required": false
  }
}
```

**Verwendung:** Wenn mehrere Optionen gleichzeitig gewählt werden können.

### 9. **Radio** (Optionsfeld)
```json
{
  "type": "radio",
  "key": "geschlecht",
  "label": "Geschlecht",
  "input": true,
  "values": [
    {"label": "Männlich", "value": "m"},
    {"label": "Weiblich", "value": "w"},
    {"label": "Divers", "value": "d"}
  ],
  "validate": {
    "required": false
  }
}
```

**Verwendung:** Für exklusive Auswahloptionen.

### 10. **Button** (Schaltfläche)
```json
{
  "type": "button",
  "key": "submit",
  "label": "Submit",
  "action": "submit",
  "input": true,
  "theme": "primary",
  "size": "md",
  "block": false,
  "disableOnInvalid": true
}
```

**Verwendung:** Für Aktionen wie Absenden, Zurücksetzen, etc.

**Action-Typen:**
- `submit` - Formular absenden
- `reset` - Formular zurücksetzen
- `event` - Benutzerdefiniertes Event auslösen

### 11. **DateTime** (Datum/Zeit)
```json
{
  "type": "datetime",
  "key": "datum",
  "label": "Datum",
  "input": true,
  "format": "yyyy-MM-dd",
  "enableTime": false,
  "enableDate": true,
  "datePicker": {
    "minDate": null,
    "maxDate": null,
    "disable": [],
    "disableWeekends": false,
    "disableWeekdays": false
  },
  "validate": {
    "required": false
  }
}
```

**Verwendung:** Für Datumsauswahl, Zeitstempel.

### 12. **Day** (Datumsfeld)
```json
{
  "type": "day",
  "key": "geburtsdatum",
  "label": "Geburtsdatum",
  "input": true,
  "fields": {
    "day": {
      "hide": false,
      "placeholder": "Tag"
    },
    "month": {
      "hide": false,
      "placeholder": "Monat"
    },
    "year": {
      "hide": false,
      "placeholder": "Jahr"
    }
  },
  "validate": {
    "required": false
  }
}
```

**Verwendung:** Für strukturierte Datumseingabe mit separaten Feldern.

### 13. **Time** (Uhrzeit)
```json
{
  "type": "time",
  "key": "uhrzeit",
  "label": "Uhrzeit",
  "input": true,
  "format": "HH:mm",
  "validate": {
    "required": false
  }
}
```

**Verwendung:** Für reine Zeiteingabe ohne Datum.

### 14. **Currency** (Währung)
```json
{
  "type": "currency",
  "key": "kosten",
  "label": "Kosten",
  "input": true,
  "currency": "EUR",
  "delimiter": true,
  "decimalLimit": 2,
  "validate": {
    "required": false,
    "min": 0
  }
}
```

**Verwendung:** Für Geldbeträge mit automatischer Formatierung.

### 15. **Panel** (Container)
```json
{
  "type": "panel",
  "key": "panel1",
  "title": "Panel Titel",
  "label": "Panel",
  "input": false,
  "collapsible": false,
  "hidden": false,
  "components": []
}
```

**Verwendung:** Zum Gruppieren von Komponenten, visuelle Strukturierung.

### 16. **Columns** (Spalten-Layout)
```json
{
  "type": "columns",
  "key": "columns1",
  "input": false,
  "columns": [
    {
      "width": 6,
      "offset": 0,
      "push": 0,
      "pull": 0,
      "size": "md",
      "components": []
    },
    {
      "width": 6,
      "offset": 0,
      "push": 0,
      "pull": 0,
      "size": "md",
      "components": []
    }
  ]
}
```

**Verwendung:** Für mehrspaltige Layouts (Bootstrap Grid System, 12 Spalten).

---

## Layout-System

### Row/Columns System
Das Layout basiert auf dem Bootstrap Grid System mit 12 Spalten.

```json
{
  "type": "columns",
  "key": "twoColumns",
  "input": false,
  "columns": [
    {
      "width": 6,
      "offset": 0,
      "size": "md",
      "components": [
        {
          "type": "textfield",
          "key": "vorname",
          "label": "Vorname"
        }
      ]
    },
    {
      "width": 6,
      "offset": 0,
      "size": "md",
      "components": [
        {
          "type": "textfield",
          "key": "nachname",
          "label": "Nachname"
        }
      ]
    }
  ]
}
```

**Spalten-Größen:**
- `width`: 1-12 (Anzahl der Spalten)
- `offset`: 0-11 (Links-Versatz)
- `size`: `xs`, `sm`, `md`, `lg`, `xl` (Responsive Breakpoints)

**Häufige Layouts:**
- **2 Spalten:** width: 6 + width: 6
- **3 Spalten:** width: 4 + width: 4 + width: 4
- **1/3 + 2/3:** width: 4 + width: 8

---

## Validierungsregeln

### Standard-Validierung

```json
{
  "validate": {
    "required": true,
    "custom": "",
    "customPrivate": false,
    "strictDateValidation": false,
    "multiple": false,
    "unique": false
  }
}
```

### Textfeld-Validierung
```json
{
  "validate": {
    "required": true,
    "minLength": 3,
    "maxLength": 50,
    "pattern": "^[A-Za-zäöüÄÖÜß\\s-]+$",
    "customMessage": "Bitte geben Sie einen gültigen Namen ein"
  }
}
```

### Zahlen-Validierung
```json
{
  "validate": {
    "required": true,
    "min": 0,
    "max": 1000000,
    "step": 0.01
  }
}
```

### Benutzerdefinierte Validierung
```json
{
  "validate": {
    "custom": "valid = (input > 0 && input < 100) ? true : 'Der Wert muss zwischen 0 und 100 liegen';"
  }
}
```

### Regex-Pattern Beispiele

**E-Mail:**
```regex
^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$
```

**Telefonnummer (Deutschland):**
```regex
^(\+49|0)[1-9][0-9]{1,14}$
```

**PLZ (Deutschland):**
```regex
^\d{5}$
```

**IBAN:**
```regex
^[A-Z]{2}[0-9]{2}[A-Z0-9]{1,30}$
```

**Datum (YYYY-MM-DD):**
```regex
^\d{4}-\d{2}-\d{2}$
```

**Nur Buchstaben (inkl. Umlaute):**
```regex
^[A-Za-zäöüÄÖÜß\s-]+$
```

**Alphanumerisch:**
```regex
^[A-Za-z0-9]+$
```

**Personalausweisnummer:**
```regex
^[A-Z]{1}[0-9]{9}$
```

**Steuernummer:**
```regex
^\d{2}\/\d{3}\/\d{5}$
```

---

## Bedingte Anzeige

### Simple Conditional (Show/Hide)
```json
{
  "conditional": {
    "show": true,
    "when": "andereFeldKey",
    "eq": "erwarteterWert"
  }
}
```

**Beispiel:**
```json
{
  "type": "radio",
  "key": "hatFuehrerschein",
  "label": "Haben Sie einen Führerschein?",
  "values": [
    {"label": "Ja", "value": "ja"},
    {"label": "Nein", "value": "nein"}
  ]
},
{
  "type": "textfield",
  "key": "fuehrerscheinnummer",
  "label": "Führerscheinnummer",
  "conditional": {
    "show": true,
    "when": "hatFuehrerschein",
    "eq": "ja"
  }
}
```

### JSON Logic Conditional
Für komplexere Bedingungen:

```json
{
  "conditional": {
    "json": {
      "and": [
        {"==": [{"var": "data.alter"}, 18]},
        {"==": [{"var": "data.zustimmung"}, true]}
      ]
    }
  }
}
```

**Operatoren:**
- `==` - Gleich
- `!=` - Ungleich
- `>`, `<`, `>=`, `<=` - Vergleiche
- `and`, `or` - Logische Verknüpfungen
- `!` - Negation

### Custom Conditional (JavaScript)
```json
{
  "customConditional": "show = data.betrag > 1000 && data.genehmigt === true;"
}
```

---

## Namenskonventionen

### Key-Naming
**Best Practices:**
- Verwende **camelCase**: `vorname`, `nachname`, `emailAdresse`
- Keine Leerzeichen oder Sonderzeichen
- Sprechende Namen: `geburtsdatum` statt `gd`
- Präfixe für Gruppierung: `adresse_strasse`, `adresse_plz`, `adresse_ort`
- Keine Umlaute im Key: `emailAdresse` ✓, `e-Mail-Adresse` ✗

**Verbotene Zeichen:**
- Leerzeichen
- Bindestriche (außer in zusammengesetzten Keys)
- Punkte (werden als Objekt-Pfad interpretiert)
- Sonderzeichen (@, #, $, %, etc.)

### Label-Naming
**Best Practices:**
- Klar und verständlich
- Mit Umlauten erlaubt: "Geburtsdatum", "E-Mail-Adresse"
- Einheitliche Terminologie
- Optionale Felder kennzeichnen: "Telefonnummer (optional)"

### ID-Generierung
- IDs werden automatisch generiert
- Format: alphanumerisch, z.B. `ekyo9cq`, `enocv3d`
- Müssen innerhalb des Formulars eindeutig sein

---

## Vollständiges Beispiel

### Dienstreiseantrag-Formular

```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "name": "Dienstreiseantrag",
  "author": "67261fc6-3ce1-4000-a782-b97e88b37698",
  "lastEditor": "67261fc6-3ce1-4000-a782-b97e88b37698",
  "creationDate": "2024-01-15T10:30:00.000Z",
  "lastModificationDate": "2024-01-15T10:30:00.000Z",
  "definition": "{\"formioFormDefinition\":{\"display\":\"form\",\"components\":[{\"type\":\"panel\",\"key\":\"antragstellerPanel\",\"title\":\"Antragsteller\",\"input\":false,\"components\":[{\"type\":\"columns\",\"key\":\"personalienColumns\",\"input\":false,\"columns\":[{\"width\":6,\"offset\":0,\"size\":\"md\",\"components\":[{\"type\":\"textfield\",\"key\":\"vorname\",\"label\":\"Vorname\",\"placeholder\":\"Max\",\"input\":true,\"validate\":{\"required\":true,\"minLength\":2,\"maxLength\":50,\"pattern\":\"^[A-Za-zäöüÄÖÜß\\\\s-]+$\"}}]},{\"width\":6,\"offset\":0,\"size\":\"md\",\"components\":[{\"type\":\"textfield\",\"key\":\"nachname\",\"label\":\"Nachname\",\"placeholder\":\"Mustermann\",\"input\":true,\"validate\":{\"required\":true,\"minLength\":2,\"maxLength\":50,\"pattern\":\"^[A-Za-zäöüÄÖÜß\\\\s-]+$\"}}]}]},{\"type\":\"email\",\"key\":\"email\",\"label\":\"E-Mail-Adresse\",\"placeholder\":\"max.mustermann@example.com\",\"input\":true,\"validate\":{\"required\":true}},{\"type\":\"textfield\",\"key\":\"personalnummer\",\"label\":\"Personalnummer\",\"placeholder\":\"12345\",\"input\":true,\"validate\":{\"required\":true,\"pattern\":\"^[0-9]{5}$\"}}]},{\"type\":\"panel\",\"key\":\"reisedetailsPanel\",\"title\":\"Reisedetails\",\"input\":false,\"components\":[{\"type\":\"textfield\",\"key\":\"reiseziel\",\"label\":\"Reiseziel\",\"placeholder\":\"Berlin\",\"input\":true,\"validate\":{\"required\":true}},{\"type\":\"columns\",\"key\":\"reisezeitraumColumns\",\"input\":false,\"columns\":[{\"width\":6,\"offset\":0,\"size\":\"md\",\"components\":[{\"type\":\"datetime\",\"key\":\"reisebeginn\",\"label\":\"Reisebeginn\",\"input\":true,\"format\":\"yyyy-MM-dd\",\"enableTime\":false,\"validate\":{\"required\":true}}]},{\"width\":6,\"offset\":0,\"size\":\"md\",\"components\":[{\"type\":\"datetime\",\"key\":\"reiseende\",\"label\":\"Reiseende\",\"input\":true,\"format\":\"yyyy-MM-dd\",\"enableTime\":false,\"validate\":{\"required\":true,\"custom\":\"valid = (new Date(input) >= new Date(data.reisebeginn)) || 'Das Reiseende muss nach dem Reisebeginn liegen';\"}}]}]},{\"type\":\"textarea\",\"key\":\"reisezweck\",\"label\":\"Zweck der Dienstreise\",\"placeholder\":\"Bitte beschreiben Sie den Zweck der Reise\",\"rows\":4,\"input\":true,\"validate\":{\"required\":true,\"minLength\":10,\"maxLength\":500}}]},{\"type\":\"panel\",\"key\":\"kostenPanel\",\"title\":\"Geschätzte Kosten\",\"input\":false,\"components\":[{\"type\":\"currency\",\"key\":\"reisekosten\",\"label\":\"Reisekosten (Fahrt/Flug)\",\"currency\":\"EUR\",\"input\":true,\"validate\":{\"required\":true,\"min\":0}},{\"type\":\"currency\",\"key\":\"unterkunftskosten\",\"label\":\"Unterkunftskosten\",\"currency\":\"EUR\",\"input\":true,\"validate\":{\"required\":false,\"min\":0}},{\"type\":\"currency\",\"key\":\"verpflegungskosten\",\"label\":\"Verpflegungskosten\",\"currency\":\"EUR\",\"input\":true,\"validate\":{\"required\":false,\"min\":0}},{\"type\":\"number\",\"key\":\"gesamtkosten\",\"label\":\"Gesamtkosten\",\"disabled\":true,\"calculateValue\":\"value = (Number(data.reisekosten || 0) + Number(data.unterkunftskosten || 0) + Number(data.verpflegungskosten || 0)).toFixed(2);\",\"input\":true}]},{\"type\":\"panel\",\"key\":\"zusatzinformationenPanel\",\"title\":\"Zusatzinformationen\",\"input\":false,\"collapsible\":true,\"components\":[{\"type\":\"radio\",\"key\":\"verkehrsmittel\",\"label\":\"Verkehrsmittel\",\"values\":[{\"label\":\"Bahn\",\"value\":\"bahn\"},{\"label\":\"Flugzeug\",\"value\":\"flugzeug\"},{\"label\":\"PKW (privat)\",\"value\":\"pkw_privat\"},{\"label\":\"PKW (Dienst)\",\"value\":\"pkw_dienst\"},{\"label\":\"Sonstiges\",\"value\":\"sonstiges\"}],\"input\":true,\"validate\":{\"required\":true}},{\"type\":\"textfield\",\"key\":\"sonstigesVerkehrsmittel\",\"label\":\"Bitte spezifizieren\",\"input\":true,\"conditional\":{\"show\":true,\"when\":\"verkehrsmittel\",\"eq\":\"sonstiges\"}},{\"type\":\"checkbox\",\"key\":\"vorschussErforderlich\",\"label\":\"Vorschuss erforderlich\",\"input\":true},{\"type\":\"currency\",\"key\":\"vorschussbetrag\",\"label\":\"Vorschussbetrag\",\"currency\":\"EUR\",\"input\":true,\"conditional\":{\"show\":true,\"when\":\"vorschussErforderlich\",\"eq\":true},\"validate\":{\"required\":true,\"min\":0}},{\"type\":\"textarea\",\"key\":\"bemerkungen\",\"label\":\"Bemerkungen\",\"rows\":3,\"input\":true}]},{\"type\":\"button\",\"key\":\"submit\",\"label\":\"Antrag absenden\",\"action\":\"submit\",\"theme\":\"primary\",\"disableOnInvalid\":true,\"input\":true}]},\"customCss\":\"\",\"dvfDefVersion\":\"1.0\"}",
  "_links": {
    "edit": {"href": "/dforms/ui/forms/550e8400-e29b-41d4-a716-446655440000/edit"},
    "self": {"href": "/dforms/api/forms/550e8400-e29b-41d4-a716-446655440000"},
    "view": {"href": "/dforms/ui/forms/550e8400-e29b-41d4-a716-446655440000/view"}
  }
}
```

**Das Beispiel zeigt:**
- ✓ Strukturierung mit Panels
- ✓ Mehrspaltige Layouts (Columns)
- ✓ Verschiedene Eingabetypen
- ✓ Validierungen (Pattern, Min/Max, Required)
- ✓ Bedingte Felder (Conditional)
- ✓ Berechnete Felder (calculateValue)
- ✓ Currency-Formatierung
- ✓ Collapsible Sections

---

## Häufige Fehler

### 1. Fehlende Required-Properties

**❌ Falsch:**
```json
{
  "type": "textfield",
  "label": "Name"
}
```

**✓ Richtig:**
```json
{
  "type": "textfield",
  "key": "name",
  "label": "Name",
  "input": true
}
```

**Erklärung:** Jede Komponente benötigt mindestens `type`, `key`, `label` und `input`.

---

### 2. Ungültige Key-Namen

**❌ Falsch:**
```json
{
  "key": "E-Mail Adresse",
  "key": "benutzer.name",
  "key": "straße"
}
```

**✓ Richtig:**
```json
{
  "key": "emailAdresse",
  "key": "benutzerName",
  "key": "strasse"
}
```

**Erklärung:** Keys dürfen keine Leerzeichen, Punkte oder Umlaute enthalten.

---

### 3. Doppelte Keys

**❌ Falsch:**
```json
{
  "components": [
    {"type": "textfield", "key": "name", "label": "Vorname"},
    {"type": "textfield", "key": "name", "label": "Nachname"}
  ]
}
```

**✓ Richtig:**
```json
{
  "components": [
    {"type": "textfield", "key": "vorname", "label": "Vorname"},
    {"type": "textfield", "key": "nachname", "label": "Nachname"}
  ]
}
```

**Erklärung:** Jeder Key muss innerhalb des Formulars eindeutig sein.

---

### 4. Falsche Spalten-Breite

**❌ Falsch:**
```json
{
  "columns": [
    {"width": 6},
    {"width": 8}
  ]
}
```

**✓ Richtig:**
```json
{
  "columns": [
    {"width": 6, "components": []},
    {"width": 6, "components": []}
  ]
}
```

**Erklärung:** Die Summe der Spaltenbreiten sollte 12 ergeben. Jede Spalte benötigt ein `components`-Array.

---

### 5. Fehlende Validation bei Required-Feldern

**❌ Falsch:**
```json
{
  "type": "textfield",
  "key": "email",
  "label": "E-Mail"
}
```

**✓ Richtig:**
```json
{
  "type": "email",
  "key": "email",
  "label": "E-Mail",
  "input": true,
  "validate": {
    "required": true
  }
}
```

**Erklärung:** Nutze den richtigen Typ (`email`) und setze `validate.required` explizit.

---

### 6. Falsche Conditional-Syntax

**❌ Falsch:**
```json
{
  "conditional": {
    "show": "feldName == 'wert'"
  }
}
```

**✓ Richtig:**
```json
{
  "conditional": {
    "show": true,
    "when": "feldName",
    "eq": "wert"
  }
}
```

**Erklärung:** Verwende die korrekte Syntax mit `show`, `when` und `eq`.

---

### 7. Ungültige Regex-Pattern

**❌ Falsch:**
```json
{
  "validate": {
    "pattern": "[0-9]+"
  }
}
```

**✓ Richtig:**
```json
{
  "validate": {
    "pattern": "^[0-9]+$"
  }
}
```

**Erklärung:** Regex-Pattern sollten mit `^` beginnen und mit `$` enden für vollständige Übereinstimmung.

---

### 8. Fehlende Input-Property

**❌ Falsch:**
```json
{
  "type": "textfield",
  "key": "name",
  "label": "Name"
}
```

**✓ Richtig:**
```json
{
  "type": "textfield",
  "key": "name",
  "label": "Name",
  "input": true
}
```

**Erklärung:** Die `input`-Property muss auf `true` gesetzt werden für Eingabefelder.

---

### 9. Falsche DateTime-Konfiguration

**❌ Falsch:**
```json
{
  "type": "datetime",
  "key": "datum",
  "format": "DD.MM.YYYY"
}
```

**✓ Richtig:**
```json
{
  "type": "datetime",
  "key": "datum",
  "label": "Datum",
  "input": true,
  "format": "yyyy-MM-dd",
  "enableTime": false,
  "enableDate": true
}
```

**Erklärung:** Verwende das korrekte Format-Pattern und setze `enableTime`/`enableDate`.

---

### 10. Panel ohne Components

**❌ Falsch:**
```json
{
  "type": "panel",
  "key": "panel1",
  "title": "Mein Panel"
}
```

**✓ Richtig:**
```json
{
  "type": "panel",
  "key": "panel1",
  "title": "Mein Panel",
  "input": false,
  "components": [
    {
      "type": "textfield",
      "key": "feld1",
      "label": "Feld 1",
      "input": true
    }
  ]
}
```

**Erklärung:** Panels benötigen ein `components`-Array, auch wenn es leer ist.

---

## Best Practices

### 1. **Strukturierung**
- Nutze Panels zur logischen Gruppierung
- Verwende Columns für übersichtliche Layouts
- Halte Formulare übersichtlich (max. 20-25 Felder pro Seite)

### 2. **Validierung**
- Setze sinnvolle Min/Max-Werte
- Verwende spezifische Pattern für Eingabeformate
- Gib aussagekräftige Fehlermeldungen an

### 3. **Benutzerfreundlichkeit**
- Nutze Placeholders für Beispielwerte
- Markiere optionale Felder explizit
- Verwende Tooltips für Erklärungen
- Nutze Collapsible Panels für Zusatzinformationen

### 4. **Performance**
- Vermeide zu viele bedingte Felder
- Nutze calculateValue nur wenn nötig
- Halte Regex-Pattern einfach

### 5. **Wartbarkeit**
- Verwende sprechende Key-Namen
- Dokumentiere komplexe Validierungen
- Halte die Struktur konsistent

---

## Template-Checkliste

Bevor Sie ein Formular importieren:

- [ ] Alle Keys sind eindeutig
- [ ] Keine ungültigen Zeichen in Keys
- [ ] `input: true` bei allen Eingabefeldern gesetzt
- [ ] Required-Felder haben entsprechende Validierung
- [ ] Spaltenbreiten ergeben 12 in Columns
- [ ] Conditional-Felder haben korrekten `when`-Referenz
- [ ] Regex-Pattern mit `^` und `$` versehen
- [ ] Button mit `action: "submit"` vorhanden
- [ ] JSON ist valide (keine Syntaxfehler)
- [ ] `dvfDefVersion` ist gesetzt

---

## Weitere Komponenten-Eigenschaften

### Gemeinsame Eigenschaften (alle Komponenten)

```json
{
  "key": "eindeutigerKey",
  "label": "Beschriftung",
  "labelPosition": "top",
  "placeholder": "Platzhalter",
  "description": "Beschreibungstext unter dem Feld",
  "tooltip": "Tooltip-Text",
  "prefix": "Präfix",
  "suffix": "Suffix",
  "customClass": "css-klasse",
  "tabindex": "",
  "hidden": false,
  "hideLabel": false,
  "disabled": false,
  "autofocus": false,
  "tableView": true,
  "modalEdit": false,
  "protected": false,
  "persistent": true,
  "multiple": false,
  "defaultValue": null,
  "clearOnHide": true,
  "refreshOn": "",
  "redrawOn": "",
  "dbIndex": false,
  "encrypted": false,
  "allowCalculateOverride": false,
  "calculateValue": "",
  "calculateServer": false
}
```

### Styling-Optionen

```json
{
  "customClass": "form-control-lg",
  "labelWidth": "",
  "labelMargin": "",
  "theme": "primary"
}
```

**Verfügbare Themes:**
- `primary` - Blau
- `secondary` - Grau
- `success` - Grün
- `danger` - Rot
- `warning` - Gelb
- `info` - Hellblau

---

## Versionierung

- **dvfDefVersion:** `1.0` (aktuelle Version)
- **Form.io Kompatibilität:** 4.x+

---

## Ressourcen

- d.velop Process Studio Dokumentation
- Form.io Component Documentation
- JSON Logic Dokumentation
- Bootstrap Grid System

---

**Erstellt am:** 2024-01-15  
**Version:** 1.0  
**Autor:** GovModeler ProcessStudio Team
