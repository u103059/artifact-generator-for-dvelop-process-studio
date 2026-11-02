# Artifact Generator for d.velop Process Studio

KI-gestützte Erstellung von Prozessmodellen, Formularen und weiteren Dateien für **d.velop Process Studio** mit GitHub Copilot, spezialisierten Agenten und wiederverwendbaren Vorlagen.

## Projektziel
Dieses Repository unterstützt dabei, fachliche Anforderungen in technische Dateien für d.velop Process Studio zu überführen. Natürlichsprachliche Beschreibungen werden mithilfe generativer KI in strukturierte Prozessmodelle und Formulare übersetzt. Vorlagen, dokumentierte Konventionen und Beispiele bilden die Grundlage für einen nachvollziehbaren und wiederverwendbaren Erstellungsprozess.

Das Projekt entwickelt den Ansatz von **GovModeler** weiter. Der ursprüngliche Schwerpunkt liegt auf Verwaltungsprozessen und ihrer standardisierten Beschreibung. Die technische Ausrichtung auf d.velop Process Studio steht nun im Mittelpunkt; der Ansatz ist grundsätzlich auch auf andere organisatorische Anwendungsbereiche übertragbar.

## Was bedeutet „Artifact“?
Ein **Artefakt** ist ein konkretes Arbeitsergebnis, das anschließend geprüft, bearbeitet und weiterverwendet werden kann. In diesem Projekt sind damit insbesondere Dateien gemeint: BPMN-Prozessmodelle, JSON-Formulare, DMN-Entscheidungsregeln und perspektivisch Skripte.

Der Begriff bezeichnet das erzeugte Ergebnis, nicht dessen Freigabestatus. Ein erstelltes Artefakt ist deshalb nicht automatisch fachlich korrekt, erfolgreich importiert oder unmittelbar produktiv einsetzbar.

## Funktionsumfang und Entwicklungsstand
| Bereich | Inhalt und aktueller Schwerpunkt |
|---|---|
| **Prozessmodelle (BPMN)** | Dokumentierter Agentenworkflow zur Erstellung von Prozessbeschreibungen und BPMN-Dateien auf Basis einer Process-Studio-spezifischen XML-Vorlage. |
| **Formulare (JSON)** | Dokumentierter Agentenworkflow zur Erstellung von Formularen und ihrer Verknüpfung mit BPMN-Benutzeraufgaben. |
| **Entscheidungsregeln (DMN)** | Agentenworkflow zur Erstellung von Entscheidungstabellen auf Basis einer aus einem Modeler-Export abgeleiteten DMN-Vorlage. Ungetestete DMN-Funktionen und die BPMN-Einbindung sind in der Zielumgebung gesondert zu prüfen. |
| **Skripte** | Perspektivischer Erweiterungsbereich, etwa für Datenverarbeitung und Serviceanbindungen; ein entsprechender Erstellungsworkflow ist bislang nicht dokumentiert. |

## Arbeitsweise
Die Erstellung erfolgt in **Visual Studio Code mit GitHub Copilot**. Spezialisierte Agenten verarbeiten die Anforderungen und orientieren sich an den hinterlegten Vorlagen und Beispielen. Das Repository stellt dafür Anweisungen und Referenzmaterial bereit, keinen eigenständigen Generator mit garantierter Ausgabevalidierung.

1. **Anforderungen beschreiben:** Prozess, Beteiligte, benötigte Daten und gewünschte Ergebnisse benennen.
2. **Fachlichen Entwurf prüfen:** Ablauf und Inhalte auf Vollständigkeit und Richtigkeit kontrollieren.
3. **Dateien erstellen:** Den passenden Agenten mit der technischen Umsetzung anhand der Vorlagen beauftragen.
4. **Ergebnisse überprüfen:** Dateistruktur, Referenzen, Variablen und fachliche Logik prüfen.
5. **Im Zielsystem erproben:** Dateien übernehmen, umgebungsspezifische Einstellungen ergänzen und das Zusammenspiel testen.

Die Dateierstellung ist von Import, Bereitstellung und Ausführung getrennt. Eine automatische Übertragung oder Synchronisierung mit d.velop Process Studio ist in den dokumentierten Agentenworkflows nicht eingerichtet.

## Prozessmodelle erstellen
Der Agent **BPMN_ProcessStudio** erstellt zunächst eine strukturierte Prozessbeschreibung mit Prozesszweck, Akteuren, Startereignis, Aktivitäten und Endereignis. Nach Bestätigung wird daraus anhand der BPMN-Vorlage eine BPMN-Datei erzeugt.

Der dokumentierte Workflow konzentriert sich auf den **sequenziellen Standardablauf („Happy Path“)**. Alternativpfade, Fehlerfälle und parallele Abläufe sind nicht Bestandteil dieses Agentenauftrags. Die Vorlage enthält darüber hinaus technische Muster, beispielsweise für Variablen und Serviceaufrufe.

Beispielauftrag: „Beschreibe den Prozess ‚Stammdaten ändern‘ als sequenziellen Standardablauf. Erstelle nach meiner Bestätigung eine BPMN-Datei für d.velop Process Studio anhand der hinterlegten Vorlage.“

## Formulare erstellen und verknüpfen
Der Agent **Form_ProcessStudio** erstellt JSON-Formulare auf Grundlage der Formularvorlage. Benötigt werden insbesondere Formularname, Zweck, Felder sowie die zugehörige BPMN-Datei und Benutzeraufgabe. Ergänzend können Validierungsregeln, bedingte Anzeigen, Berechnungen und Layoutwünsche angegeben werden.

Der Workflow umfasst die Erstellung einer Formular-ID und deren Eintragung als `camunda:formKey` an der zugehörigen Benutzeraufgabe. Die Formularstruktur basiert auf Form.io und dem im Projekt dokumentierten d.velop-spezifischen JSON-Rahmen. Die tatsächliche Verfügbarkeit des Formulars im Zielsystem muss anschließend sichergestellt werden.

Beispielauftrag: „Erstelle ein Formular für die Aufgabe ‚Antrag prüfen‘ im vorhandenen BPMN-Modell. Benötigt werden eine schreibgeschützte Antragsnummer, eine verpflichtende Entscheidung und ein Kommentarfeld. Verknüpfe das Formular mit der Benutzeraufgabe.“

## Entscheidungstabellen erstellen
Der Agent **DMN_ProcessStudio** erstellt DMN-Entscheidungstabellen auf Grundlage von `DMN.Template.md`. Benötigt werden Tabellenname, Eingabe- und Ausgabevariablen mit Typ (`String`, `Number`, `Boolean`, `Identity`) sowie die Regeln. Der Agent legt zunächst einen Tabellenentwurf zur Bestätigung vor und erzeugt danach die Datei in `CreateDMN/`.

Die Vorlage kennzeichnet verifizierte und ungetestete Inhalte. Die Einbindung in BPMN erfolgt nur, wenn eine exportierte Referenz-BPMN mit Geschäftsregelaktivität in `example-DMN/` vorliegt.

Beispielauftrag: „Erstelle eine Entscheidungstabelle ‚Standort zu Prüfer‘: Eingabe `Standort_GR` (String), Ausgabe `Pruefer_GR` (Identity). Intern → Benutzer A, extern → Benutzer B, sonst → Gruppe Prüfer.“

## Voraussetzungen
- **Visual Studio Code**, **GitHub Copilot** und die für die Agentennutzung erforderliche Berechtigung.
- Zugriff auf die zum jeweiligen Workflow gehörenden Agentenkonfigurationen, Vorlagen und Beispiele.
- Geeignete BPMN-, JSON- und gegebenenfalls DMN-Editoren zur Prüfung und Nachbearbeitung.
- Zugang zu einer geeigneten **d.velop-Process-Studio-Testumgebung**, um Import und Nutzung zu überprüfen.

Die technische Dokumentation beschreibt die bisherige Entwicklungsumgebung und empfohlene Erweiterungen. Versionsangaben und Editor-Empfehlungen sind für die eigene Umgebung zu prüfen.

## Aufbau des Repositorys
| Ablage | Zweck |
|---|---|
| CreateBPMN | Erstellte BPMN-Prozessmodelle. |
| CreateForm | Erstellte Formulardateien im JSON-Format. |
| CreateDMN | Erstellte DMN-Entscheidungstabellen. |
| example-DMN | DMN-Beispiele aus dem d.velop process modeler als technische Referenz. |
| BeispielForms | Formularbeispiele als technische Referenz. |
| example-bpmn-processes | BPMN-Beispiele unter anderem für Gateways, Timer, Mehrfachausführungen und Serviceaufrufe. |

Ausführliche Beschreibungen enthalten die Dokumentation des BPMN-Agenten und die Dokumentation des Formularagenten.

## Qualität, Sicherheit und Verantwortung
Generierte Dateien sind **prüfpflichtige Entwürfe**. Vor der Verwendung sind insbesondere Syntax, fachlicher Ablauf, Rollen, Berechtigungen, Prozessvariablen, Formularreferenzen und externe Dienste zu kontrollieren. Die Kennzeichnung eines BPMN-Modells als ausführbar ersetzt keinen erfolgreichen Import- und Laufzeittest.

Verwende für Entwicklung und Beispiele möglichst synthetische Daten. Personenbezogene oder vertrauliche Informationen dürfen nur entsprechend den geltenden Datenschutz-, Sicherheits- und Organisationsvorgaben verarbeitet werden. KI-Ausgaben ersetzen keine fachliche oder rechtliche Prüfung.

## Weiterentwicklung und Beiträge
Willkommen sind zusätzliche Vorlagen, verbesserte Agentenanweisungen, dokumentierte Fehlerkorrekturen und nachvollziehbare Testergebnisse. Entwicklungsziele sind insbesondere die Erweiterung der DMN-Unterstützung, geeignete Workflows für Skripte sowie eine stärkere automatisierte Validierung der erzeugten Dateien.

Bei Kompatibilitätsmeldungen sollten die verwendete Zielversion, erforderliche Konfigurationen und tatsächlich durchgeführte Prüfungen angegeben werden. Das Repository ist ein eigenständiges Projekt zur Unterstützung der Arbeit mit d.velop Process Studio, kein offizielles Produkt von d.velop.
