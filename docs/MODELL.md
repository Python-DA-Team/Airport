# Modellnachweis aus der veröffentlichten PBIX-Datei

Git-Blob der geprüften PBIX: `85e80da240ba29d7bead3e8e96cde4d643eab1fb`. Stand der Dokumentation: 15.11.2026.

Die folgenden DAX- und Power-Query-Ausdrücke wurden aus dem eingebetteten Modell der veröffentlichten PBIX gelesen. Nur absolute lokale Excel-Pfade sind durch einen Platzhalter ersetzt. Die Ausdrücke sind ein lesbarer Modellnachweis, kein eigenständig ausführbares PBIP-Projekt.

## DAX-Kennzahlen

### Calendar / Total Flights

```dax
VAR MapSelection =
    ISFILTERED(FlightPaths[Path ID])
        || ISFILTERED(FlightPaths[Location])
VAR SelectedOrigins = VALUES(FlightPaths[Airport])
VAR SelectedDestinations = VALUES(FlightPaths[Destination])
RETURN
    IF(
        MapSelection,
        CALCULATE(
            COUNTROWS(Flights_Raw),
            TREATAS(SelectedOrigins, Flights_Raw[Airport]),
            TREATAS(SelectedDestinations, Flights_Raw[Destination])
        ),
        COUNTROWS(Flights_Raw)
    )
```

### Calendar / Total Passengers

```dax
VAR MapSelection =
    ISFILTERED(FlightPaths[Path ID])
        || ISFILTERED(FlightPaths[Location])
VAR SelectedOrigins = VALUES(FlightPaths[Airport])
VAR SelectedDestinations = VALUES(FlightPaths[Destination])
RETURN
    IF(
        MapSelection,
        CALCULATE(
            SUM(Flights_Raw[Passengers]),
            TREATAS(SelectedOrigins, Flights_Raw[Airport]),
            TREATAS(SelectedDestinations, Flights_Raw[Destination])
        ),
        SUM(Flights_Raw[Passengers])
    )
```

### Calendar / Total Revenue

```dax
VAR MapSelection =
    ISFILTERED(FlightPaths[Path ID])
        || ISFILTERED(FlightPaths[Location])
VAR SelectedOrigins = VALUES(FlightPaths[Airport])
VAR SelectedDestinations = VALUES(FlightPaths[Destination])
RETURN
    IF(
        MapSelection,
        CALCULATE(
            SUM(Flights_Raw[Revenue_EUR]),
            TREATAS(SelectedOrigins, Flights_Raw[Airport]),
            TREATAS(SelectedDestinations, Flights_Raw[Destination])
        ),
        SUM(Flights_Raw[Revenue_EUR])
    )
```

### Calendar / Average Delay

```dax
VAR MapSelection =
    ISFILTERED(FlightPaths[Path ID])
        || ISFILTERED(FlightPaths[Location])
VAR SelectedOrigins = VALUES(FlightPaths[Airport])
VAR SelectedDestinations = VALUES(FlightPaths[Destination])
RETURN
    IF(
        MapSelection,
        CALCULATE(
            AVERAGE(Flights_Raw[Delay_Min]),
            TREATAS(SelectedOrigins, Flights_Raw[Airport]),
            TREATAS(SelectedDestinations, Flights_Raw[Destination])
        ),
        AVERAGE(Flights_Raw[Delay_Min])
    )
```

### Calendar / Average Satisfaction

```dax
VAR MapSelection =
    ISFILTERED(FlightPaths[Path ID])
        || ISFILTERED(FlightPaths[Location])
VAR SelectedOrigins = VALUES(FlightPaths[Airport])
VAR SelectedDestinations = VALUES(FlightPaths[Destination])
RETURN
    IF(
        MapSelection,
        CALCULATE(
            AVERAGE(Flights_Raw[Satisfaction]),
            TREATAS(SelectedOrigins, Flights_Raw[Airport]),
            TREATAS(SelectedDestinations, Flights_Raw[Destination])
        ),
        AVERAGE(Flights_Raw[Satisfaction])
    )
```

### Calendar / Delayed Flights

```dax
CALCULATE(
    [Total Flights],
    Flights_Raw[Status] = "Delayed"
)
```

### Calendar / On-Time %

```dax
DIVIDE(
    CALCULATE(
        [Total Flights],
        Flights_Raw[Status] = "On Time"
    ),
    [Total Flights]
)
```

### Calendar / Delayed %

```dax
DIVIDE(
    [Delayed Flights],
    [Total Flights]
)
```

### FlightPaths / Route Flights

```dax
VAR RouteOrigin = SELECTEDVALUE(FlightPaths[Airport])
VAR RouteDestination = SELECTEDVALUE(FlightPaths[Destination])
RETURN
    CALCULATE(
        [Total Flights],
        TREATAS({ RouteOrigin }, Flights_Raw[Airport]),
        TREATAS({ RouteDestination }, Flights_Raw[Destination])
    )
```

### FlightPaths / Route Passengers

```dax
VAR RouteOrigin = SELECTEDVALUE(FlightPaths[Airport])
VAR RouteDestination = SELECTEDVALUE(FlightPaths[Destination])
RETURN
    CALCULATE(
        [Total Passengers],
        TREATAS({ RouteOrigin }, Flights_Raw[Airport]),
        TREATAS({ RouteDestination }, Flights_Raw[Destination])
    )
```

### Calendar / Dashboard Guidance

```dax
"Select an airport or click a route to filter all KPIs"
```

## Berechnete Tabellen

### Calendar

```dax
ADDCOLUMNS(
    CALENDAR(
        MIN(Flights_Raw[Date]),
        MAX(Flights_Raw[Date])
    ),
    "Year", YEAR([Date]),
    "MonthNum", MONTH([Date]),
    "Month", FORMAT([Date], "MMMM"),
    "YearMonth", FORMAT([Date], "YYYY-MM")
)
```

### Airports

```dax
DISTINCT(
    SELECTCOLUMNS(
        Flights_Raw,
        "Airport", Flights_Raw[Airport]
    )
)
```

### FlightPaths

```dax
VAR Routes =
    SUMMARIZE(
        Flights_Raw,
        Flights_Raw[Airport],
        Flights_Raw[Destination]
    )
RETURN
    UNION(
        SELECTCOLUMNS(
            Routes,
            "Path ID", [Airport] & " → " & [Destination],
            "Point Order", 0,
            "Location", [Airport],
            "Airport", [Airport],
            "Destination", [Destination]
        ),
        SELECTCOLUMNS(
            Routes,
            "Path ID", [Airport] & " → " & [Destination],
            "Point Order", 1,
            "Location", [Destination],
            "Airport", [Airport],
            "Destination", [Destination]
        )
    )
```

## Power Query

### Flights_Raw

```powerquery
let
    Джерело = Excel.Workbook(File.Contents("<lokaler Datenordner>\Airport_Data.xlsx"), null, true),
    Flights_Raw_Sheet = Джерело{[Item="Flights_Raw",Kind="Sheet"]}[Data],
    #"Заголовки з підвищеним рівнем" = Table.PromoteHeaders(Flights_Raw_Sheet, [PromoteAllScalars=true]),
    #"Змінений тип" = Table.TransformColumnTypes(#"Заголовки з підвищеним рівнем",{{"Date", type date}, {"Flight", type text}, {"Airline", type text}, {"Airport", type text}, {"Destination", type text}, {"Passengers", Int64.Type}, {"Delay_Min", Int64.Type}, {"Status", type text}, {"Revenue_EUR", type number}, {"Satisfaction", type number}}),
    #"Усі слова з великих букв" = Table.TransformColumns(#"Змінений тип",{{"Status", Text.Proper, type text}}),
    #"Обітнутий текст" = Table.TransformColumns(#"Усі слова з великих букв",{{"Airline", Text.Trim, type text}}),
    #"Обітнутий текст1" = Table.TransformColumns(#"Обітнутий текст",{{"Flight", Text.Trim, type text}, {"Airline", Text.Trim, type text}, {"Airport", Text.Trim, type text}, {"Destination", Text.Trim, type text}, {"Status", Text.Trim, type text}}),
    #"Усі слова з великих букв1" = Table.TransformColumns(#"Обітнутий текст1",{{"Airport", Text.Proper, type text}})
in
    #"Усі слова з великих букв1"
```

## Beziehungen

| Von | Nach | Aktiv | Kardinalität | Filterrichtung |
| --- | --- | --- | --- | --- |
| Flights_Raw.Date | Calendar.Date | Ja | n:1 | Einfach |
| Flights_Raw.Airport | Airports.Airport | Ja | n:1 | Einfach |
| FlightPaths.Airport | Airports.Airport | Ja | n:1 | Einfach |
