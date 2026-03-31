# death_checker_cpp

Checks a list of clients against [rip.ie](https://rip.ie) death notices and flags any deceased individuals. Takes a CSV of names and locations, queries the Rip.ie API for each person, and outputs a results CSV.

---

## Run It

`Win + R` → type `cmd` → OK, then paste:

```cmd
C:\Users\darra\death_checker_cpp\rip_checker.exe C:\Users\darra\Downloads\your_clients.csv
```

Output saves automatically to `Downloads\results_your_clients.csv`.

---

## Common Commands

**Basic run:**
```cmd
C:\Users\darra\death_checker_cpp\rip_checker.exe C:\Users\darra\Downloads\clients.csv
```

**Custom output file:**
```cmd
C:\Users\darra\death_checker_cpp\rip_checker.exe C:\Users\darra\Downloads\clients.csv C:\Users\darra\Downloads\results.csv
```

**Slower rate if getting blocked:**
```cmd
C:\Users\darra\death_checker_cpp\rip_checker.exe C:\Users\darra\Downloads\clients.csv --rate 2000
```

**Only check notices from a given year onwards:**
```cmd
C:\Users\darra\death_checker_cpp\rip_checker.exe C:\Users\darra\Downloads\clients.csv --from-year 2020
```

> If you don't pass `--from-year`, you'll be prompted at startup: `Search death notices from 2020 onwards only? [y/N]`. Press `y` to filter from 2020, or Enter to search all years.

---

## CSV Format

Only `FirstName` and `LastName` (or equivalents) are required. Column names are detected automatically and are case-insensitive.

| Column | Aliases accepted | Required |
|--------|-----------------|----------|
| FirstName | Forename, First Name, Given Name | Yes |
| LastName | Surname, Family Name | Yes |
| County | Region, Area, Location, Address | Recommended |
| Town | Townland, City, Village | Optional |
| ClientID | ID, No, Ref | Optional |

Example:
```csv
Surname,Firstname,Town,County
MURPHY,John,Milltown,Kerry
KELLY,Patrick,Galway,Galway
```

---

## Output Columns

The results CSV (`Downloads\results_<inputname>.csv`) contains:

| Column | What it means |
|--------|---------------|
| `ClientID` | ID from input CSV, or row number if absent |
| `FirstName` / `LastName` | As read from input |
| `County` | County from input CSV |
| `Status` | `Alive`, `Deceased`, `Possible Match`, or `Data Not Found` |
| `Address` | Town + county from the death notice |
| `Detail` | Notice date and location |
| `URL` | Direct link to the rip.ie death notice (blank if Alive) |

### Status values

| Status | Meaning |
|--------|---------|
| **Alive** | Search completed — no matching death notice found |
| **Deceased** | Death notice found — name and county confirmed |
| **Possible Match** | Name matches but county differs — needs manual review |
| **Data Not Found** | Network error or missing name fields |

---

## Things to Know

- **Nicknames work** — Paddy/Patrick, Gerry/Gerard, Mickey/Michael, Padraig/Patrick, etc.
- **Fadas handled** — Sean matches Seán, O'Brien matches Ó'Brien
- **Female notices skipped** — won't return false matches for female names
- **Maiden names handled** — `Andrews (née Kavanagh)` searches both surnames
- **Year filter is client-side** — all pages are fetched from Rip.ie and filtered locally
- **Output file locked?** — close the results CSV in Excel before running again

---

## Rebuild

If you need to recompile after a source code change, open Command Prompt (`Win + R` → `cmd`) and paste:

```cmd
"C:\Program Files (x86)\Microsoft Visual Studio\18\BuildTools\VC\Auxiliary\Build\vcvarsall.bat" x64 && cmake --build C:\Users\darra\death_checker_cpp\build --config Release
```

The rebuilt exe is copied automatically to `C:\Users\darra\death_checker_cpp\rip_checker.exe`.

### Full reconfigure (only needed if CMakeLists.txt changes)

```cmd
"C:\Program Files (x86)\Microsoft Visual Studio\18\BuildTools\VC\Auxiliary\Build\vcvarsall.bat" x64 && cmake --preset windows-default -S C:\Users\darra\death_checker_cpp && cmake --build C:\Users\darra\death_checker_cpp\build --config Release
```

> Always use a plain Command Prompt (`Win + R` → `cmd`) — not a Developer Command Prompt or PowerShell, as those can have conflicting environment variables.
