# LSPath
Lists full filepaths matching a search pattern with optional command execution on each file.

About     | Current Release
----------|-----------------------
Version   | 2.3
Date      | July 29, 2023
Platforms | Windows, macOS, Linux

# Installation

1. Install [Morlock](https://morlock.sh).
2. `morlock install brombres/lspath`

# Usage

    lspath <options> [filepath1 filepath2 ...]

## Options

### `--command=<command-line>` `-c <command-line>`
Invokes a system() call instead of printing result filepaths. The command
can contain one of two kinds of placeholder marker:

Placeholder Marker | Description
-------------------|--------------------------------------------------------------
`$`                | Command is invoked N times for N result filepaths, with a different filepath substituted each time.
`$$`               | Command is invoked once and `$$` is replaced by a space-separated list of every filepath.

Single quotes must be used around the command to prevent `$` being escaped by the shell. See also: --quiet

Example (macOS/Linux)                       | Description
--------------------------------------------|-------------------------------------------------------
`lspath "**/*.rogue" --command='ls -l "$"'` | Long listing of each filepath.
`lspath "**/*.rogue" -e -c 'wc -l $$'`      | Word count of all filepaths, escaping spaces.

### `--esc` `-e`
Paths are displayed with spaces and most other symbols escaped. For example
a filepath containing `abc 123` would display as `abc\ 123`.

### `--grep=<pattern>` `-g <pattern>`
Only prints filepaths of files that contain one or more given patterns.
Despite the name these are "wildcard" patterns, not classic "grep" regular
expressions. Patterns are applied line by line and are not case sensitive.
Example:

    lspath --grep="a*z"

prints filepaths of all files containing a line that starts with `a` and
ends with `z`. Specifying multiple grep patterns requires that a file contain
all patterns in order to have its filepath printed.

### `--files`
When searching for matching `--name` patterns, only the filename is checked, not the folder path.

### `--folders`
When searching for matching `--name` patterns, only the folder path is checked, not the filename.

### `--help` `-h` `-?`
Print this help text.

### `--hidden` `-a`
Show all hidden files - hidden files are omitted by default.

### `--name=<name>` `-n <name>`
Only print filepaths containing the given name pattern. Name comparisions
are case-insensitive. If multiple `--name` directives are given, each
filepath need only match one of the names to be printed. Wildcard names
patterns may be used, e.g. `"ABC*.cpp"`.

### `--quiet, -q`
Prevents the `--command` option from displaying each command before executing it.
Does not suppress the execution output. Useful when piping the result of `lspath`
into another command.

### `--recursive, -r`
All specified folders are processed recursively.

## Wildcard Patterns
Put patterns in quotes to ensure that LSTree's non-standard wildcard patterns
are correctly processed. By example:

Pattern         | Description
----------------|-------------------------------------------
`"*"`           |All files in current folder
`"**"`          |All files, recursively
`"**/*.rogue"`  |All .rogue files, recursively
`"A?E"`         |All 3-letter files in current folder starting with `A` and ending with `E`.

# Example

## Show filepaths for all .cpp, and .h files

    lspath "**/*.cpp" "**/*.h"

