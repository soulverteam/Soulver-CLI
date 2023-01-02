# Soulver-CLI

This is a free standalone command-line version of [Soulver](https://soulver.app). It's a great alternative to bc for your terminal.

This command line tool supports all the same calculation features at the Mac & iPad versions, including unit conversions, live currency conversions, date & time math & all supported natural language functions.

See Soulver's [documentation](https://documentation.soulver.app) for all supported syntaxes and features.

### System Requirements

Requires macOS 12 or later. 

Both Intel & Apple Silicon Macs are supported and the tool has been notarized for Gatekeeper.

### Installation

`soulver-cli` is available via [Homebrew](https://brew.sh) or as a downloadable binary from the [releases page][] or inside this repository.

```
$ brew install soulver-cli
```

On first launch this tool automatically downloads a ~1MB package file entitled `SoulverCore_SoulverCore.bundle`. It contains json files required to bootstrap the calculation engine's features (unit names, natural function names, city names for timezone conversions, etc).

This bundle must exist in the same directory as the `soulver` executable for calculation to be performed.

### Interactive mode

Type `soulver` from Terminal (with no arguments) to open an interactive calculation mode (like bc):

```
$ soulver
Welcome to Soulver
>>> 32% of $340k
$108800.00
>>> March 12 + 4 weeks 5 days
14 April
>>> 98 USD in JPY
¥12832
```

Variable declarations are supported:

```
>>> bill total = $128.89
$128.89
>>> bill total / 5 people
$25.78
```

### Direct arguments

You can launch soulver with a direct argument to be evaluated:

```
$ soulver "new timestamp"
1672646020
```

Or via a pipe
```
$ echo "123 + 456" | soulver
579
```

If you specify a file path as the direct argument each line will be evaluated.

```
$ soulver myFile.txt
```

### Live currency rates

An installation of the [Mac version of Soulver](https://soulver.app) is __not__ required for this command line tool to function, with one exception: this tool does not refresh the latest currency rates on its own.

If you want the latest currency rates to be accessible to this tool, the Mac version is required to keep currency rates up to date. 
