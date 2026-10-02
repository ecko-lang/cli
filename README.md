# cli - Ecko Std Lib Package

Declarative command-line parsing: describe the program as a map, get back its
options, arguments, subcommand and `--help` request, and render usage text
from the same description.

Pure computation - no capabilities. From 1.0 this is the `std.cli` module from
the Ecko standard library, moved into a package: the same spec, the same
result and the same help text.

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
import std.os

spec = {
    name: "greet",
    about: "print a friendly greeting",
    options: [
        { name: "loud", short: "l", flag: true, help: "shout it" },
        { name: "count", short: "n", default: 1, help: "repeat N times" },
    ],
    args: [{ name: "name", required: true }],
}

r = cli.parse(spec, os.args())
if r.help {
    print(cli.help(spec, r.command))
}
r.options.count    # an Int, because the default is one
r.args.name
```

Subcommands are a `commands` list of specs; the first positional picks one, and
`r.command` names it:

```ecko
spec = { name: "vcs", commands: [{ name: "add", args: [{ name: "path" }] }, { name: "commit" }] }
r = cli.parse(spec, ["add", "src/"])    # r.command == "add", r.args.path == "src/"
```

## API

| function | result |
|---|---|
| `parse(spec, argv)` | `{ options, args, rest, command, help }`. Raises `{ kind: "cli", message }` on bad input. |
| `help(spec, command?)` | Usage text for the spec, or for one subcommand. `null` is the top level, so `help(spec, r.command)` works directly. |
| `parser(name, description?)`, `flag(spec, name, help?)`, `opt(spec, name, opts?)`, `arg(spec, name, opts?)` | The 0.x builder: each returns the spec with one more entry, so they chain with `\|>`. |
| `usage(spec)` | `help(spec)`, under its 0.x name. |

A spec is `{ name, about?, options?, args?, commands? }`.

- **An option** is `{ name, short?, flag?, default?, required?, help? }`.
  `--name value`, `--name=value`, `-s value` and `-svalue` all work. A `flag`
  takes no value and is `false` unless given. The value is an Int or a Float
  when the `default` is one, and a String otherwise. Every option is present in
  the result, holding its default when absent.
- **An argument** is `{ name, required?, help? }`, bound in order. Positionals
  beyond the declared ones, and everything after `--`, go to `rest`.
- **`--help` or `-h`** sets `help` and skips validation, so a missing required
  value cannot hide it.

Errors, all `kind: "cli"`: an unknown option (`'--nope'` or `'-z'`), an option
that needs a value, a value that is not the default's type, a missing required
option or argument, no command, or an unknown command.

## Notes

- **Moving from `std.cli`:** replace `import std.cli` with `import cli` after
  `ecko get`. Results and help text are identical - the package was checked
  against the native module on 51 spec and argument combinations, on Ecko 0.57
  and 0.58. The one difference: errors are `kind: "cli"`, where the native
  module raised the generic `"bug"`.
- **Moving from cli 0.x:** `parse` now returns `{ options, args, rest, ... }`
  rather than one flat map, so `r.loud` becomes `r.options.loud` and `r.name`
  becomes `r.args.name`. The builder functions still work and build the 1.0
  spec. `opt`'s `kind` is gone: the type follows the `default`. `required` on
  an option, which 0.x documented and ignored, now works.

## Testing

```bash
ecko test
```

Offline and deterministic. The expected results, help text included, are what
the native `std.cli` returns for the same input.

## License

MIT
