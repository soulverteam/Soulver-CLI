# Soulver CLI

This is a free standalone command-line version of [Soulver](https://soulver.app). It's a great alternative to [bc](https://pubs.opengroup.org/onlinepubs/9699919799/utilities/bc.html) for your terminal.

This command line tool supports all the same calculation features at the Mac & iPad versions, including unit conversions, live currency conversions, date & time math & all supported natural language functions.

See Soulver's [documentation](https://documentation.soulver.app) for all supported syntaxes and features.

### System Requirements

Requires macOS 12 or later. 

Both Intel & Apple Silicon Macs are supported. The tool is notarized and compatible with Gatekeeper.

### Installation

The easiest way to install the Soulver CLI is with [Homebrew](https://brew.sh):

```
brew install soulver-cli
```

The Soulver CLI is also available as a downloadable binary from the Github releases page for this repository (or by cloning this repository).

On first launch the tool will automatically download a ~1MB package file titled `SoulverCore_SoulverCore.bundle` to the same directory from which the tool is run. The bundle must remain in the same directory as the tool.

The bundle contains json files required to bootstrap the calculation engine and support its various features (unit names, natural function names, city names for timezone conversions, etc). 
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
