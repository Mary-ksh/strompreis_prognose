# Strompreis-Prognose Deutschland ⚡

Ein End-to-End-Projekt mit maschinellem Lernen: Ich sage den **Day-ahead-Strompreis** für die Gebotszone **DE-LU** (Deutschland/Luxemburg) für den nächsten Tag voraus und vergleiche mein Modell ehrlich mit einer einfachen Regel (Baseline).

> **Stand:** Schritt 2 von 6 ist fertig. Die Daten werden geladen und geprüft. Modell und Ergebnisse folgen.

## Ziel

- Den Day-ahead-Preis für morgen vorhersagen.
- Prüfen, ob ein ML-Modell wirklich besser ist als eine einfache Regel („gleiche Stunde in der Vorwoche“).
- Den ganzen Weg zeigen: Daten holen → speichern → analysieren → Modell → Auswertung.

## Fahrplan

- [x] 1. Grundgerüst: GitHub-Repository, README, virtuelle Umgebung
- [x] 2. Daten holen: ein Jahr Day-ahead-Preise von der Energy-Charts API
- [ ] 3. SQLite-Datenbank und SQL-Abfragen (Durchschnittspreis pro Stunde und pro Wochentag)
- [ ] 4. Baseline „gleiche Stunde in der Vorwoche“ und Fehler messen (MAE)
- [ ] 5. scikit-learn-Modell (z. B. Random Forest), Trainings- und Testdaten zeitlich getrennt, Vergleich mit der Baseline
- [ ] 6. Ergebnisse und Grenzen dokumentieren

Später geplant: Wind- und Sonnendaten, Streamlit-Dashboard, Docker, GitHub Actions.

## Daten

| | |
|---|---|
| Quelle | [Energy-Charts API](https://api.energy-charts.info) (Fraunhofer ISE), Endpunkt `/v2/price` |
| Gebotszone | DE-LU |
| Zeitraum | November 2025 bis Oktober 2026 (12 volle Monate) |
| Auflösung | 15 Minuten (96 Werte pro Tag) |
| Einheit | EUR/MWh |

**Gut zu wissen:**

- Seit dem 1. Oktober 2025 gibt es in der europäischen Day-ahead-Auktion 15-Minuten-Produkte statt Stundenprodukten. Der gewählte Zeitraum liegt komplett danach.
- An den Tagen der Zeitumstellung hat ein Tag 92 (März) oder 100 (Oktober) Viertelstunden. Deshalb speichert das Projekt alle Zeiten zusätzlich in UTC.
- Preise können negativ sein, zum Beispiel bei viel Wind und Sonne.

### Lizenz der Daten

Die Preisdaten für DE-LU stehen unter **CC BY 4.0**. Quelle: Bundesnetzagentur | SMARD.de, bereitgestellt über Energy-Charts (Fraunhofer ISE).

Die Daten liegen **nicht** im Repository. Man lädt sie mit dem Skript neu (siehe [Nutzung](#nutzung)).

## Projektstruktur

```
strompreis_prognose/
├── fetch_prices_year.py   # lädt ein Jahr Preise und speichert sie als CSV
├── requirements.txt       # benötigte Python-Pakete
├── .env.example           # Vorlage für die Datei .env
├── .gitignore
├── README.md
└── data/                  # wird vom Skript angelegt, nicht im Repository
    └── raw/
        ├── json/          # Cache: eine JSON-Datei pro Monat
        └── prices_DE-LU_2025-11_bis_2026-10.csv
```

## Installation

Voraussetzungen: **Python 3.14** und Git.

Windows (PowerShell):

```powershell
git clone https://github.com/Mary-ksh/strompreis_prognose.git
cd strompreis_prognose
py -3.14 -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
Copy-Item .env.example .env
```

macOS/Linux:

```bash
git clone https://github.com/Mary-ksh/strompreis_prognose.git
cd strompreis_prognose
python3.14 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
cp .env.example .env
```

## Nutzung

```bash
python fetch_prices_year.py
```

Das Skript

1. teilt den Zeitraum in 12 Monate auf und holt jeden Monat einzeln,
2. wartet zwischen zwei Anfragen 35 Sekunden, weil die API für Preise nur 2 Anfragen pro Minute erlaubt,
3. speichert jede Antwort als JSON-Datei (Cache). Bei einem neuen Start werden gespeicherte Monate nicht noch einmal geladen,
4. prüft die Daten (fehlende Werte, Lücken, doppelte Zeitstempel, Werte pro Tag) und speichert sie als CSV.

Der erste Lauf dauert etwa 7 Minuten, danach nur wenige Sekunden.

### Aufbau der CSV-Datei

| Spalte | Bedeutung |
|---|---|
| `timestamp_utc` | Beginn der Viertelstunde in UTC |
| `timestamp_local` | Beginn der Viertelstunde in deutscher Zeit (mit Sommer-/Winterzeit) |
| `price_eur_mwh` | Day-ahead-Preis in EUR/MWh |

## Ergebnisse

Folgen nach Schritt 4 und 5.