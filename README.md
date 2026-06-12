# Soulver CLI

A standalone command-line version of [Soulver](https://soulver.app) — the notepad calculator — for macOS.

Evaluate quick expressions, run a live interactive session, or read, write, and manipulate your Soulver sheetbooks from the terminal and from scripts.

```sh
$ soulver "25% off 159.99 + 10% tax"
131.99

$ soulver "98 USD in EUR"
€84.67

$ echo "5 days from now" | soulver
17 June 2026
```

## Install

```sh
brew install --cask soulver-cli
```

The Homebrew token stays `soulver-cli` (a bare `soulver` would mean the Mac app); the installed command is just `soulver`.

Or download the latest signed, notarized binary from the [Releases](https://github.com/soulverteam/Soulver-CLI/releases) page and place `soulver` somewhere on your `PATH`.

If you have the Soulver app installed, you can also install the bundled CLI from **Soulver → Install Command Line Tool…**, which symlinks the very same tool into `/usr/local/bin`.

Currency and exchange rates are fetched live on first use and cached locally (`~/.config/soulver/currency-rates.json`), so conversions work offline after the first run and refresh automatically.

## Quick calculations

Pass an expression as an argument, or pipe it in:

```sh
soulver "1,234.50 * 8% APR over 3 years"
soulver "12 feet in cm"
soulver "time in tokyo when it is 9am in new york"
echo "150 GBP + 200 USD in AUD" | soulver
```

Give it a file and every line is evaluated:

```sh
soulver ~/Desktop/budget.txt
```

Use `--locale` for locale-aware parsing and formatting:

```sh
soulver --locale de_DE "1.000,50 + 1"   # 1001,5
```

## Interactive mode

Run `soulver` with no arguments to enter an interactive session. Unlike a plain
`bc`-style calculator, it's a stateful Soulver notepad — **variables and line
references persist** across the session:

```
$ soulver
Soulver — interactive calculator. Type 'exit' or press Ctrl-D to quit.
>>> rent = 2400
2400
>>> rent * 12
28800
>>> line1 + 500
2900
>>> 1 BTC in USD
US$63,238.43
>>> exit
```

Press `Ctrl-D` or type `exit` to quit.

## Working with sheetbooks

The CLI can read and edit the same `.sheetbook` files the Soulver app uses. By
default it finds your active sheetbook automatically; set an explicit default with
`soulver use`, override per-command with `--path`, or via the `SOULVER_SHEETBOOK`
environment variable.

```sh
soulver use ~/Documents/Budget.sheetbook   # set a default sheetbook
```

### Sheets

```sh
soulver sheet list                          # list all sheets
soulver sheet show "Budget"                 # print a sheet's contents
soulver sheet create "July Expenses"        # add a new sheet
soulver sheet duplicate "Budget"
soulver sheet move "Budget" --to "2026"     # move into a folder
soulver sheet pin "Budget"
soulver sheet archive "Old Notes"
soulver sheet delete "Scratch"
soulver sheet save-as "Budget" --output ~/Budget.slvr
```

### Lines

```sh
soulver line list   --sheet "Budget"
soulver line append --sheet "Budget" "Rent: 2400"
soulver line insert --sheet "Budget" --at 3 "Utilities: 180"
soulver line set    --sheet "Budget" 2 "Rent: 2500"          # set line 2
soulver line mark   subtotal --sheet "Budget" 4             # mark line 4 as a subtotal
soulver line format --sheet "Budget" 1 --thousands-separators  # format line 1
soulver line copy   answer --sheet "Budget" --line 5       # output line 5's answer
soulver line delete --sheet "Budget" 6                     # delete line 6
```

Line behaviours for `mark`: `expression`, `subtotal`, `timepoint`, `running-total`, `running-budget`. Formatting flags for `format`: `--thousands-separators` / `--no-thousands-separators`, `--notation` / `--no-notation`, `--precision <n>`, `--alignment left|center|right`.

Append multiple lines from stdin:

```sh
printf 'Coffee: 4.50\nLunch: 14\nTaxi: 22' | soulver line append --sheet "Today"
```

### Global variables & definitions

```sh
soulver variable list
soulver variable set hourly_rate "85 USD"
soulver variable delete hourly_rate

soulver definition show
soulver definition append "story = 3.5 m"     # define a custom unit
```

### Search

```sh
soulver search "rent"                    # across all sheets
soulver search "total" --sheet "Budget"  # within one sheet
soulver search "500" --json
```

### Export

```sh
soulver export txt  --sheet "Budget" --output budget.txt
soulver export csv  --sheet "Budget" --output budget.csv
soulver export html --sheet "Budget" --show-line-numbers --show-total --output budget.html
soulver export png  --sheet "Budget" --width 800 --output budget.png
soulver export txt  --sheet "Budget" --clipboard          # straight to the clipboard
```

### Create new files

```sh
soulver new sheetbook --path ~/Documents/Finances.sheetbook
soulver new sheet --path ~/calc.slvr
```

### Inspect (for tooling & agents)

```sh
soulver inspect line --sheet "Budget" --line 1 --json
```

## Scripting & agents

The CLI is designed to be driven by scripts, editor plugins, and AI agents:

- **`--json` on reads** — `sheet list`, `line list`, `search`, `inspect`, `variable list`, etc. return structured JSON.
- **`--json` on writes** — create/append/set/mark/delete report what changed, so you can chain commands without scraping prose:

  ```sh
  id=$(soulver sheet create "Budget" --json | jq -r .id)
  line=$(soulver line append --sheet "$id" "Rent: 2400" --json | jq -r .line)
  ```

- **Errors** go to stderr and the process exits non-zero, so failures are easy to detect.
- **Bare expressions** evaluate directly: `soulver "1 + 1"` (no subcommand needed).

```sh
soulver sheet list --json
soulver line list --sheet "Budget" --json
soulver inspect line --sheet "Budget" --line 1 --json
```

## Alfred

Soulver ships an [Alfred](https://www.alfredapp.com) workflow that calls the
bundled CLI directly — type a calculation into Alfred and press Return to copy the
answer. The workflow is included with the Soulver app.

## Help

```sh
soulver --help
soulver <command> --help
soulver help <command> <subcommand>
```

---

Soulver CLI is built on [SoulverCore](https://github.com/soulverteam/SoulverCore),
the calculation engine behind Soulver.
