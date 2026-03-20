# death_checker_cpp — Personal Reference

A C++ command-line tool that checks a list of clients against [rip.ie](https://rip.ie) death notices and flags any deceased individuals.

---

## Run It

Open Command Prompt, then:

```cmd
cd C:\Users\darra\death_checker_cpp\build
```

```cmd
death_checker.exe C:\Users\darra\Downloads\your_clients.csv
```

Output saves automatically as `results_your_clients.csv` in your Downloads folder.

---

## Common Commands

**Basic run:**
```cmd
death_checker.exe C:\Users\darra\Downloads\clients.csv
```

**Custom output file:**
```cmd
death_checker.exe C:\Users\darra\Downloads\clients.csv C:\Users\darra\Downloads\results.csv
```

**Slower rate if getting blocked:**
```cmd
death_checker.exe C:\Users\darra\Downloads\clients.csv --rate 2000
```

**Only check notices from 2020 onwards:**
```cmd
death_checker.exe C:\Users\darra\Downloads\clients.csv --from-year 2020
```

> If you don't pass `--from-year`, you'll be prompted at startup: `Search death notices from 2020 onwards only? [y/N]`. Answer `y` to apply the 2020 filter, or press Enter to search all years.

---

## CSV Format

Must have these columns (names are case-insensitive):

| Column | Required |
|--------|----------|
| `Surname` or `LastName` | Yes |
| `Firstname` or `FirstName` | Yes |
| `County` | Recommended |
| `Town` | Optional |

Example:
```csv
Surname,Firstname,Town,County
MURPHY,John,Dublin,Dublin
KELLY,Patrick,Galway,Galway
```

---

## Output Columns

| Column | What it means |
|--------|---------------|
| `Status` | `Alive`, `Deceased`, `Possible Match`, or `Data Not Found` |
| `Address` | Location from the death notice (blank if Alive) |
| `Detail` | Full detail — notice date and location |
| `URL` | Direct link to the rip.ie death notice (blank if Alive) |

---

## Things to Know

- **County is strict** — if the county doesn't match the death notice, returns `Alive`. No cross-county guesses.
- **Nicknames work** — Paddy/Patrick, Gerry/Gerard, Mickey/Michael, Padraig/Patrick etc.
- **Fadas handled** — Sean matches Seán, O'Brien matches Ó'Brien
- **Female notices skipped** — won't return false matches for female names
- **Year filter is client-side** — all pages are still fetched from rip.ie; records before the cutoff year are discarded locally. For common surnames this means up to 30 pages are always fetched regardless of the filter.
- **Output file locked?** — close the results CSV in Excel, then run again

---

## Build / Rebuild

Open a regular Command Prompt and run this from the build folder:

```cmd
cd C:\Users\darra\death_checker_cpp\build
```

```cmd
"C:\Program Files (x86)\Microsoft Visual Studio\18\BuildTools\VC\Auxiliary\Build\vcvarsall.bat" x64 && "C:\Program Files (x86)\Microsoft Visual Studio\18\BuildTools\Common7\IDE\CommonExtensions\Microsoft\CMake\Ninja\ninja.exe"
```

> **Important:** Do NOT use a regular `ninja` command or the Developer Command Prompt shortcut — the full command above is required to load the correct VS environment before building. Without it you'll get `LNK1181: cannot open input file 'ws2_32.lib'`..
