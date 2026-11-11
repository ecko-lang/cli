# cli

Declarative command-line parsing: describe the program as a map, get back its
options, arguments, subcommand and `--help` request, and render usage text
from the same description. Pure computation - no capabilities.

## Install

```bash
ecko get github.com/ecko-lang/cli
```

```ecko
import cli
```

## Usage

```ecko
import cli

spec = {
    name: "greet",
    about: "print a friendly greeting",
    options: [
        { name: "loud", short: "l", flag: true, help: "shout it" },
        { name: "count", short: "n", default: 1, help: "repeat N times" },
    ],
    args: [{ name: "name", required: true, help: "who to greet" }],
}

# A real program passes os.args(); a fixed list keeps this one deterministic.
r = cli.parse(spec, ["-ln", "2", "Ada"])
print(r.options)        # {count: 2, loud: true}
print(r.args.name)      # Ada

try {
    cli.parse(spec, ["--count", "lots", "Ada"])
} catch (e) {
    print(e.reason)     # bad_value
    print(e.message)    # option '--count' expects a whole number, got 'lots'
}

print(cli.help(spec))
# greet - print a friendly greeting
#
# Usage: greet [options] <name>
#
# Options:
#   -l, --loud    shout it
#   -n, --count <value>    repeat N times (default: 1)
#       --help    show this help
#
# Arguments:
#   <name>    who to greet (required)
```

### A real program

Show the help when asked, and the error plus the help on bad input:

```ecko norun
import cli
import std.os

spec = { name: "greet", args: [{ name: "name", required: true }] }

r = try { cli.parse(spec, os.args()) } catch (e) {
    if get(e, "kind") != "cli" { error(e) }
    print("error: {e.message}\n")
    print(cli.help(spec))
    os.exit(2)
}
if r.help {
    print(cli.help(spec, r.command))
    os.exit(0)
}
print("Hello, {r.args.name}.")
```

### Subcommands

`commands` is a list of specs; the first positional picks one and
`r.command` names it. The top level's own options go before the command and
are returned in `options` beside the command's:

```ecko
import cli

spec = {
    name: "vcs",
    options: [{ name: "verbose", short: "v", flag: true }],
    commands: [
        { name: "add", options: [{ name: "all", short: "a", flag: true }], args: [{ name: "path" }] },
        { name: "commit", options: [{ name: "message", short: "m", required: true }] },
    ],
}

r = cli.parse(spec, ["-v", "add", "-a", "src/"])
print(r.command)        # add
print(r.options)        # {all: true, verbose: true}
print(r.args.path)      # src/
```

## API

| function | returns |
|---|---|
| `parse(spec, argv)` | `{ options, args, rest, command, help }` for the argument list `argv` |
| `help(spec, command = null)` | the usage text for the spec, or for the subcommand `command` |
| `parser(name, description = "")` | a new, empty spec, for the builder functions below |
| `flag(spec, name, help = "")` | `spec` with one more boolean flag |
| `opt(spec, name, opts = {})` | `spec` with one more option that takes a value; `opts` holds its `help`, `default`, `required`, `short` |
| `arg(spec, name, opts = {})` | `spec` with one more positional argument; `opts` holds its `help`, `required` |
| `usage(spec)` | `help(spec)`, under its 0.x name |

## Notes

### The spec

A spec is `{ name, about?, options?, args?, commands? }`.

- **An option** is `{ name, short?, flag?, default?, required?, help? }`.
  `--name value`, `--name=value`, `-s value` and `-svalue` all work, and
  short options cluster: `-vn5` is `-v -n 5`, the first option that takes a
  value consuming the rest. The value is an Int or a Float when the
  `default` is one, and a String otherwise. Every option is present in the
  result, holding its default when absent.
- **A flag** takes no value and is `false` unless given. `--name=value` on a
  flag is an error; `--no-name` sets it to `false`, which is how a flag whose
  `default` is `true` is turned off. An option literally named `no-name` is
  matched before that.
- **An argument** is `{ name, required?, help? }`, bound in order. Positionals
  beyond the declared ones, and everything after `--`, go to `rest`.
- **`--help`** sets `help` and skips validation, so a missing required value
  cannot hide it. **`-h`** does the same unless an option declares
  `short: "h"`, in which case it is that option.
- **`commands`**: a list of specs. Options of the top level are accepted
  before the command name only, as the usage line says. When a command
  declares an option of the same name, the command's wins.

### Errors

| kind | reason | when |
|---|---|---|
| `cli` | `unknown_option` | an option the spec does not declare (`--nope`, `-z`) |
| `cli` | `flag_value` | a flag given a value (`--verbose=false`) |
| `cli` | `missing_value` | an option that takes a value is last on the line |
| `cli` | `bad_value` | the value is not the type of the option's `default` |
| `cli` | `missing_option` | a `required` option is absent |
| `cli` | `missing_argument` | a `required` argument is absent |
| `cli` | `no_command` | the spec has `commands` and none was given |
| `cli` | `unknown_command` | the command, or `help`'s `command`, is not in the spec |
| `bug` | `bad_argument` | `spec` is not a map, `argv` not a list, or a builder argument has the wrong type |
| `bug` | `bad_spec` | an option's `short` is not exactly one character |

A `cli` error's `message` is written for the person at the terminal
(`unknown option '--nope'`), so print it as it stands.

### Moving from `std.cli`

Replace `import std.cli` with `import cli` after `ecko get`. Results and help
text are the same; errors are `kind: "cli"`, where the native module raised
`bug`.

## Testing

```bash
ecko test
```

## License

[MIT](LICENSE)
