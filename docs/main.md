# cli

## `parse(spec, argv)`

Parse `argv` - a list such as `os.args()` - against a spec map, returning
`{ options, args, rest, command, help }`.

The spec is `{ name, about?, options?, args?, commands? }`. Each option is
`{ name, short?, flag?, default?, required?, help? }`: `--name value`,
`--name=value`, `-s value` and `-svalue` all work, a `flag` takes no value,
and the value is an Int or Float when the `default` is one. Each arg is
`{ name, required?, help? }`, bound in order; extra positionals, and
everything after `--`, land in `rest`. `--help` or `-h` sets `help` and
skips validation. With `commands` (a list of specs), the first positional
picks one and `command` names it.

Bad input - an unknown option, a missing value, a missing required option
or argument, an unknown command - raises `{ kind: "cli", message }`.

```ecko
r = parse({ name: "x", options: [{ name: "n", default: 1 }] }, ["--n", "3"])
r.options.n    # 3
```

## `help(spec, command = null)`

Usage text for a spec: the name and `about`, a usage line, then options,
arguments and commands with their help. Pass a command name - for example
the `command` of a `parse` result - for that subcommand's help; `null`
means the top level.

```ecko
if r.help { print(help(spec, r.command)) }
```

## `parser(name, description = "")`

Start a spec, to build up with `flag`, `opt` and `arg` and pass to `parse`.
The same as writing the map `{ name, about: description, options: [], args: [] }`.

```ecko
spec = parser("greet", "say hello") |> flag("loud", "shout it")
```

## `flag(spec, name, help = "")`

Add a boolean flag, `--name`: present means true. A leading `--` on the
name is dropped.

```ecko
spec = parser("tool") |> flag("verbose", "print more")
```

## `opt(spec, name, opts = {})`

Add an option that takes a value, `--name value`. `opts` may carry `help`,
`default`, `required` and `short`; the value's type follows the default.

```ecko
spec = parser("tool") |> opt("count", { default: 1, help: "how many" })
```

## `arg(spec, name, opts = {})`

Add a positional argument, bound in declaration order. `opts` may carry
`help` and `required`.

```ecko
spec = parser("tool") |> arg("name", { required: true })
```

## `usage(spec)`

The help text for a spec - the same as `help(spec)`, under its 0.x name.
