# Changelog

## 2.0.0

### Fixed

- Combined short options are read in full: `-vn5` is `-v -n 5`. Before, every
  letter after the first was dropped without an error.
- A flag given a value (`--verbose=false`) is an error. Before, the value was
  ignored and the flag was set to `true`.
- With subcommands, the top level's options before the command are parsed and
  returned in `options` (`vcs --verbose add`, `vcs --config x.toml add`), and
  an unknown one is reported. Before, they were dropped, an option's value
  was taken for the command name, and an unknown option passed silently.
  `options` now also holds the top level's options, with their defaults.
- An option declaring `short: "h"` is reachable as `-h`; `--help` still asks
  for help. Before, `-h` always meant help.
- An option whose `short` is not one character is refused when the spec is
  parsed, rather than never matching.

### Added

- `--no-<flag>` sets a flag to `false`, so a flag with `default: true` can be
  turned off.
- Every `cli` error carries a `reason` (`unknown_option`, `flag_value`,
  `missing_value`, `bad_value`, `missing_option`, `missing_argument`,
  `no_command`, `unknown_command`) to match on instead of the message.

### Breaking

- A spec or argument list of the wrong type (`parse("x", [])`, `help(42)`,
  `opt(spec, "n", 5)`) raises `kind: "bug"` with `reason: "bad_argument"`
  instead of `kind: "cli"`. These are mistakes in the calling program, not
  bad input from its user.
  Migration: none needed for a correct program; a `catch` that turned these
  into a usage message should let them through, or match `bug` too.
