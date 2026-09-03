# Soulver CLI

A standalone command-line version of [Soulver](https://soulver.app) for macOS.

Evaluate expressions, use an interactive session, or read and edit Soulver sheetbooks from the terminal and from scripts.

```sh
$ soulver "25% off 159.99 + 10% tax"
131.99175

$ soulver "98 USD in EUR"
€84.67

$ echo "5 days from now" | soulver
28 July 2026 at 8:36 am
```

## Install

```sh
brew install --cask soulver-cli
```

The Homebrew token stays `soulver-cli` (a bare `soulver` would mean the Mac app); the installed command is just `soulver`.

Or download the latest signed, notarized binary from the [Releases](https://github.com/soulverteam/Soulver-CLI/releases) page and place `soulver` somewhere on your `PATH`.

The Soulver app also bundles a copy, installed from **Soulver → Install Command Line Tool…**. That copy runs inside the app's sandbox, so it evaluates expressions but cannot read or edit sheetbooks; the sheetbook commands below need the Homebrew or downloaded build.

Upgrading from a 1.x Homebrew install: if `soulver` exits at once naming a leftover `SoulverCore_SoulverCore.bundle` in `/opt/homebrew/bin`, delete that folder. The 1.x cask put it there and 2.x cannot use it.

## Quick calculations

Pass an expression as an argument, or pipe it in:

```sh
soulver '$1,000 for 3 years at 7% compounded quarterly'
soulver "12 feet in cm"
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

Other flags, on `soulver "…"` and `soulver eval "…"` alike:

| Flag | Effect |
|------|--------|
| `--strict` | Exit 1 on anything not understood, instead of reading past it |
| `--auto-convert` | Convert a lone result to its counterpart: `20 C` → `68 °F` |
| `--dp <n>` | Decimal places |
| `--json` | `{"ok": true, "expression": "2+2", "result": "4"}` |
| `--offline` | Never refresh currency rates |
| `--path <file>` | The sheetbook whose definition sheet and global variables apply |
| `--no-definitions` | The stock engine, no sheetbook |

Expressions are evaluated with your sheetbook's definition sheet and global variables, so a unit or variable defined in the app works here too.

### Currency rates

Rates are fetched from Soulver's rate service and cached in `~/.config/soulver/currency-rates.json`; each release also includes the rates from the day it was built. If a conversion uses rates older than 12 hours, a refresh runs in a background process and applies to the next evaluation. Evaluation never waits for the network, and expressions without a currency never use it.

```sh
soulver rates status     # rates in use and their age
soulver rates refresh    # fetch now
```

## Interactive mode

Run `soulver` with no arguments to enter an interactive session. Variables and
line references persist across the session:

```
$ soulver
>>> rent = 2400
2400
>>> rent * 12
28800
>>> line1 + 500
2900
>>> 1 BTC in USD
US$66150.61
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
soulver inspect line --sheet "Budget" 1 --json
```

## Scripting & agents

The CLI can be driven from scripts, editor plugins, and agents:

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
soulver inspect line --sheet "Budget" 1 --json
```

## Help

```sh
soulver --help
soulver <command> --help
soulver help <command> <subcommand>
```
