# CLI - Ecko Std Lib Package

Command-line argument parsing for [Ecko](https://ecko.sh), written in Ecko:
typed flags, options, positionals, defaults, and generated usage.

## Install

```bash
ecko get github.com/ecko-sh/cli
```

## Usage

```ecko
import cli
import std.os

spec = cli.parser("greet", "print a friendly greeting")
  |> cli.flag("--loud", "shout it")
  |> cli.opt("--count", { kind: "int", default: 1, help: "repeat N times" })
  |> cli.arg("name", { required: true })

args = cli.parse(spec, os.args())   # { loud: bool, count: int, name: string }
print(cli.usage(spec))              # generated help text
```

## API

| Function | Description |
|---|---|
| `parser(name, description?)` | Start a parser spec |
| `flag(spec, name, help?)` | A boolean flag (`--loud`) |
| `opt(spec, name, opts?)` | An option taking a value. `opts`: `{ kind: "string"\|"int"\|"float"\|"bool", default, help }` |
| `arg(spec, name, opts?)` | A positional. `opts`: `{ required, help }` |
| `parse(spec, argv)` | Parse a list of args → a map of values, or raise a kind-`"cli"` error |
| `usage(spec)` | Generated help text |

`flag`/`opt`/`arg` return a new spec, so they chain with `|>`. Options accept
both `--count 3` and `--count=3`. Result keys are the names without dashes.

## Testing

```bash
ecko test tests/
```

## License

MIT - see [LICENSE](LICENSE).
