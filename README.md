# ✈️ Airport Performance Dashboard

<p align="center"><strong>Power BI Portfolio · Flughafenanalyse</strong></p>
<p align="center"><em>Flugaufkommen · Passagiere · Pünktlichkeit · Flugverbindungen</em></p>
<p align="center">
  <img alt="Power BI" src="https://img.shields.io/badge/Power_BI-Desktop-F2C811?style=flat-square">
  <img alt="Power Query" src="https://img.shields.io/badge/Power_Query-M-217346?style=flat-square">
  <img alt="DAX" src="https://img.shields.io/badge/Kennzahlen-DAX-174A70?style=flat-square">
  <img alt="Lernprojekt" src="https://img.shields.io/badge/Projekt-Lernmaterial-orange?style=flat-square">
</p>

Ein Power-BI-Lernprojekt zur Auswertung von Flug- und Passagierdaten. KPI-Karten, eine Routenkarte, ein Statusdiagramm und eine monatliche Übersicht unterstützen den Vergleich von Flughäfen und die Untersuchung von Verspätungen.

**[📥 Bericht herunterladen](<Airport Performance Dashboard.pbix>)** · **[🧮 DAX, Power Query und Beziehungen](docs/MODELL.md)** · **[📂 Berichtsdefinitionen](report-source/README.md)**

> [!NOTE]
> Die Beschreibung ist auf Deutsch; die veröffentlichte PBIX verwendet englische Berichts- und Kennzahlenbeschriftungen. Es handelt sich um ein Lernprojekt, nicht um ein produktives Flughafensteuerungssystem oder eine Live-Flugverfolgung.

## 📋 Schnellnavigation

| Projekt kennenlernen | Methodik prüfen | Bericht verwenden |
| --- | --- | --- |
| [Geschäftsfragen](#geschaeftsfragen) | [Kennzahlen](#kennzahlen) | [Dateien und Start](#start) |
| [Berichtsseiten](#seiten) | [Datenmodell](#modell) | [Grenzen](#grenzen) |

<a id="geschaeftsfragen"></a>

## 🎯 Geschäftsfragen

- Wie verteilen sich Flüge und Passagieraufkommen auf die Flughäfen?
- Welcher Anteil der erfassten Flüge ist als pünktlich beziehungsweise verspätet gekennzeichnet?
- Wie hoch ist die durchschnittliche erfasste Verzögerung?
- Welche Verbindungen sind im Datensatz enthalten, und wie verteilt sich das Flugaufkommen über die Monate?

Die Auswertungen machen Unterschiede und mögliche Prüfschwerpunkte sichtbar. Eine nachgewiesene Senkung von Verspätungen oder eine wirtschaftliche Verbesserung wird nicht behauptet.

<a id="seiten"></a>

## 📊 Berichtsseiten

| Seite | Implementierte Inhalte |
| --- | --- |
| **Overview** | Flughafenauswahl, Routenkarte mit Azure Maps, KPI-Karten, Kreisringdiagramm nach Flugstatus und Bedienhinweise |
| **Monthly Trends** | Liniendiagramm mit Fluganzahl nach Monat |

Die Definitionen enthalten 12 Visualobjekte einschließlich zweier Hinweis-Karten. Die Modellformeln und Feldzuordnungen wurden aus der veröffentlichten PBIX gelesen; Kartendarstellung, Geokodierung und sämtliche Klickinteraktionen wurden bei dieser Dokumentationsprüfung nicht erneut in Power BI ausgeführt.

<a id="kennzahlen"></a>

## 📐 Kennzahlen

| Kennzahl | Implementierte Berechnung / Einordnung |
| --- | --- |
| Total Flights | Anzahl der Zeilen in `Flights_Raw`; kein DISTINCTCOUNT der Flugnummer |
| Total Passengers | Summe `Passengers` im Filterkontext |
| Average Delay | Mittelwert `Delay_Min` über vorhandene Werte; nicht auf verspätete Flüge eingeschränkt |
| Average Satisfaction | Mittelwert der vorhandenen Zufriedenheitswerte pro Datensatz, nicht nach Passagieren gewichtet |
| Delayed Flights | Fluganzahl mit Status `Delayed` |
| On-Time % | Fluganzahl mit Status `On Time` geteilt durch die gesamte Fluganzahl |
| Delayed % | Verspätete Fluganzahl geteilt durch die gesamte Fluganzahl |
| Route Flights / Route Passengers | Zuordnung von Start und Ziel über `SELECTEDVALUE` und `TREATAS` |
| Total Revenue | Summe `Revenue_EUR`; im Modell vorhanden, aber nicht als Umsatzkarte auf den veröffentlichten Seiten dargestellt |
| Dashboard Guidance | Textmaß für den Bedienhinweis |

**Modellumfang:** 11 Measures einschließlich des Textmaßes. Das ist nicht die Anzahl der sichtbaren KPI-Karten. Vollständige Ausdrücke: [MODELL.md](docs/MODELL.md).

<a id="modell"></a>

## 🗃️ Datenaufbereitung und Modell

**Technologien:** Power BI Desktop, Power Query/M, DAX; Excel als Quelldatei.

Die Power-Query-Abfrage importiert das Tabellenblatt `Flights_Raw` aus `Airport_Data.xlsx`, übernimmt die Kopfzeile, setzt Datentypen und bereinigt ausgewählte Textfelder mit Trim- und Schreibweisenfunktionen.

| Tabelle | Funktion |
| --- | --- |
| Flights_Raw | Importierte Flugdatensätze mit Datum, Flugnummer, Airline, Start, Ziel, Passagieren, Verzögerung, Status, Umsatz und Zufriedenheit |
| Calendar | Berechnete Kalendertabelle vom ersten bis zum letzten Flugdatum |
| Airports | Berechnete Liste unterschiedlicher Startflughäfen |
| FlightPaths | Berechnete Routentabelle mit Start- und Zielpunkten |

Aktive n:1-Beziehungen verbinden `Flights_Raw.Date` mit `Calendar.Date`, `Flights_Raw.Airport` mit `Airports.Airport` und `FlightPaths.Airport` mit `Airports.Airport`.

### Geprüfter Datenstand

Die eingebettete Tabelle enthält **10.000 Zeilen** vom **01.01.2025 bis 31.12.2025**. Darunter sind **7.026** Zeilen mit Status `On Time` und **2.974** mit `Delayed`. In `Delay_Min` fehlen 20 Werte, in `Satisfaction` 30 Werte. Diese Angaben stammen aus dem ausgelesenen PBIX-Datenbestand und sind keine Messung realer Betriebsleistung.

<a id="start"></a>

## 📁 Dateien und Start

```text
Airport/
├── Airport Performance Dashboard.pbix
├── README.md
├── docs/
│   └── MODELL.md
└── report-source/
    ├── README.md
    └── definition/                 # JSON-Auszüge der Seiten und Visuals
```

1. Die PBIX-Datei herunterladen und in Power BI Desktop öffnen.
2. Die Seiten `Overview` und `Monthly Trends` auswählen.
3. Für einen Datenrefresh die ursprüngliche Excel-Datei lokal bereitstellen und den absoluten Quellpfad in Power Query anpassen.

Die PBIX enthält importierte Daten. `Airport_Data.xlsx` ist derzeit nicht im Repository enthalten; ein frischer Datenimport ist mit dem Repository allein daher nicht reproduzierbar.

Der zuvor allein veröffentlichte PBIP-Verweis enthielt keine zugehörigen Report- und Modellordner. Er wurde entfernt, damit kein unvollständiges Projekt als ausführbarer Einstieg angeboten wird. `report-source/` und `docs/MODELL.md` dienen zur technischen Einsicht; sie ersetzen kein vollständiges PBIP-Paket.

<a id="grenzen"></a>

## ⚠️ Grenzen und Weiterentwicklung

- **Datengrundlage:** Lern- und Portfolioprojekt; Herkunft und Erhebungsmethode des Übungsdatensatzes sind hier nicht unabhängig belegt.
- **Fluganzahl:** Die Kennzahl setzt eine Zeile pro Flugereignis voraus. Wiederholte Flugnummern an unterschiedlichen Tagen sind damit nicht automatisch Duplikate.
- **Status und Verspätung:** Die Pünktlichkeitsquote basiert auf dem Textstatus, nicht auf einem definierten Minuten-Grenzwert.
- **Fehlwerte:** Mittelwerte ignorieren leere Werte. Die fehlenden Verzögerungs- und Zufriedenheitswerte wurden im Modell nicht vollständig ersetzt.
- **Mehrfachauswahl von Routen:** Herkunft und Ziel werden in den KPI-Maßen als getrennte Mengen übertragen. Bei Auswahl mehrerer Routen können weitere Kombinationen einbezogen werden; eine exakte Filterung ausgewählter Routenpaare ist damit nicht garantiert.
- **Monatsvergleich:** Das Diagramm verwendet den Monatsnamen. Für Daten über mehrere Jahre sollte auf Jahr-Monat umgestellt werden.
- **Azure Maps:** Darstellung und Geokodierung hängen von den verfügbaren Kartendiensten und den Power-BI-Einstellungen ab.
- **Persönlicher Beitrag:** Das Repository zeigt die technische Umsetzung. Eine detaillierte Zuordnung von Arbeitsschritten oder eine kommerzielle Projektrolle wird daraus nicht abgeleitet.

### Prüfstand

Die veröffentlichten Berichtsdefinitionen, DAX-Ausdrücke, Power-Query-Abfrage, Beziehungen und eingebetteten Flugdatensätze wurden am 15.11.2026 gelesen. Die Git-Blob-ID der unveränderten PBIX ist im [Modellnachweis](docs/MODELL.md) dokumentiert. Diese Prüfung ist kein vollständiger Lauf aller DAX-Filterkontexte.

### Nächste Schritte

- Herkunft und Nutzungsbedingungen der Quelldaten dokumentieren.
- Ein vollständiges, zur PBIX passendes PBIP-Projekt und echte Berichtsscreenshots ergänzen.
- Mehrfachauswahl von Routen sowie Status- und Verzögerungslogik fachlich prüfen.
- Nach einer Berichtübersetzung die deutsche Dokumentation und die Bildschirmansichten auf denselben Stand bringen.
