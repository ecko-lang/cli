# cli

## `parser(name, description = "")`

Start a command-line spec. Build it up with `flag`, `opt` and `arg`, then
hand it to `parse`.

```ecko
spec = parser("greet", "say hello")
```

## `flag(spec, name, help = "")`

Add a boolean flag: present on the command line means true.

```ecko
spec = flag(spec, "verbose", "print more")
```

## `opt(spec, name, opts = empty_map())`

Add a named option that takes a value (`--name value`). `opts` may carry
`help`, `default`, and `required`.

## `arg(spec, name, opts = empty_map())`

Add a positional argument, in declaration order. `opts` may carry `help`,
`default`, and `required`.

## `parse(spec, argv)`

Parse `argv` against a spec, returning a map of flags, options and
positional arguments. Raises with a usage message on unknown or missing
arguments.

## `usage(spec)`

The rendered `--help` text for a spec: description, usage line, then the
flags, options and positional arguments with their help strings.
