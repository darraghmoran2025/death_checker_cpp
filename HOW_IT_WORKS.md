# How It Works

## What it does

Reads a CSV of client names and locations, searches each person against the Rip.ie death notice database, and outputs a CSV with a status and a direct URL to any notice found.

---

## File Layout

```
death_checker_cpp/
  rip_checker.exe        — The compiled program, run this directly
  libcurl.dll            — HTTP library (must stay alongside the exe)
  zlib1.dll              — Compression library (must stay alongside the exe)
  src/
    main.cpp             — Entry point: argument parsing, main loop
    csv_reader.cpp/h     — Reads the input CSV into client records
    rip_scraper.cpp/h    — Queries Rip.ie and matches each person
    result_writer.cpp/h  — Writes the output CSV and terminal summary
```

---

## Data Flow

```
your_clients.csv
       |
       v
  CsvReader           — Parses headers and rows into a list of client records
       |
       v
  RipScraper          — Called once per client; queries Rip.ie and returns a result
       |
       v
  ResultWriter        — Prints a summary table to the terminal, writes output CSV
       |
       v
Downloads\results_your_clients.csv
```

---

## Step by Step

### 1. Reading the CSV (`csv_reader`)

Opens the input file and detects column positions from the header row. Column names are matched case-insensitively against a list of aliases — so `Surname`, `LastName`, `Family Name` all map to the same field. Only `FirstName` and `LastName` are required; `County`, `Town`, and `ClientID` are optional but improve accuracy.

Each row becomes a `ClientRecord`:
```
client_id, first_name, last_name, town, county, date_of_birth
```

Quoted CSV fields (e.g. `"Smith, Jr."`) and escaped double-quotes are handled correctly.

---

### 2. Querying Rip.ie (`rip_scraper`)

Rip.ie exposes a GraphQL API at `https://rip.ie/api/graphql`. For each client, the scraper sends a POST request searching by surname and receives back paginated records. Each record contains:

```
id, firstname, surname, createdAt, county.name, town.name
```

Up to 30 pages (~1,200 records) are fetched per search. The JSON response is parsed using a regex — no external JSON library required.

A 1,500ms delay between requests prevents being blocked (adjustable with `--rate`).

---

### 3. Matching Logic — 5 Passes

The scraper runs up to 5 passes against the fetched records and returns at the first confirmed hit:

| Pass | What it tries |
|------|---------------|
| 1 | Surname + town + county (strict) |
| 2 | Surname + county only (town on Rip.ie may differ from your CSV) |
| 3 | Maiden name, if present in surname e.g. `Andrews (née Kavanagh)` |
| 4 | Name only, no county — returns `Possible Match` for manual review |
| 5 | Full name search (`John Murphy`) as fallback for older buried notices |

---

### 4. Name Normalisation

Before any comparison, names are normalised so common variations don't cause missed matches:

| Issue | How it's handled |
|-------|-----------------|
| Irish fadas | `Seán` → `Sean`, `Ó'Brien` → `O'Brien` |
| Apostrophes | Both ASCII `'` and Rip.ie's Unicode `'` (U+2019) stripped |
| Nicknames | ~40 groups built in: Pat/Patrick/Paddy, Mick/Michael, Jim/James, etc. |
| Parentheticals | `John (Johnny)` stripped to `John` before comparing |
| County prefix | `Co. Galway`, `County Galway`, `Galway` all normalise to `galway` |
| Female names | ~100 common Irish/English female names — notices with these are skipped |

---

### 5. Output (`result_writer`)

After all clients are processed:

**Terminal** — a formatted table is printed showing every client's status and detail.

**CSV** — written to `Downloads\results_<inputname>.csv` with these columns:

```
ClientID, FirstName, LastName, County, Status, Address, Detail, URL
```

The `URL` column is a direct link to the Rip.ie notice page, e.g.:
```
https://rip.ie/death-notice/john-murphy-kerry-milltown-624712
```
It is only populated for `Deceased` and `Possible Match` results.

---

## Status Values

| Status | Meaning |
|--------|---------|
| **Alive** | Search completed — no matching death notice found |
| **Deceased** | Death notice found — name and county confirmed |
| **Possible Match** | Name matches but county differs — check the URL manually |
| **Data Not Found** | Network error or name fields missing from input |
