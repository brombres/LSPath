# LSPath
Lists full filepaths matching a search pattern with optional command execution on each file.

About     | Current Release
----------|-----------------------
Version   | 2.7
Date      | August 4, 2026
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

The following variants of `$` are supported:

Placeholder Marker | Description
-------------------|--------------------------------------------------------------
`$`                | Replaced by full filepath (folder + filename)
`$(folder)`        | Replaced by folder only
`$(filename)`      | Replaced by filename only (without folder)
`$(name)`          | Base filename only (without folder or extension)
`$(ext)`           | Replaced by filename extension only (e.g. "txt")
`$(0N)`            | Replaced by filepath listing index mapped to start at "N", with 0's defining additional minimium digits that are zero-filled. For example, `$(0)` is replaced with `0, 1, 2, ...` and `$(001)` is replaced with `001, 002, 003, ...`.

The following variants of `$$` are supported:

Placeholder Marker | Description
-------------------|--------------------------------------------------------------
`$$`               | Replaced by a space-separated list of every filepath.
`$$(...)`          | Replaced by a space-separated list of every filepath variant using the same rules as `$(...)` above.
`$$[N]`            | Replaced by the filepath at index N. 1 is the first. -1 is the last, -2 is the next-to-last, etc.
`$$[N](...)`       | Replaced by the filepath at index N using the same variants as `$(...)` above (except for `$(0N)`).
`$$[0]`            | Replaced with empty string, allowing a command to be created that runs once for 1+ files but does not list any specific files in the command.

Single quotes must be used around the command to prevent `$` being escaped by the shell. See also: --quiet

Example (macOS/Linux)                       | Description
--------------------------------------------|-------------------------------------------------------
`lspath "**/*.rogue" --command='ls -l "$"'` | Long listing of each filepath.
`lspath "**/*.rogue" -e -c 'wc -l $$'`      | Word count of all filepaths, escaping spaces.

### `--esc` `-e`
Paths containing spaces or other special characters are displayed quoted. For example
a filepath containing `abc 123` would display as `'abc 123'`. Useful when copying and
pasting results into other commands. For piping results into another command, use
`--pipe` instead.

### `--exclude=<filename>` `-x <filename>`
Exclude filenames matching the given pattern. `--exclude` overrides any matches
made with `--name`. For example, `lspath -n .png -x .import` will include
`XYZ.png` but will exclude `XYZ.png.import`. Multiple `--exclude` directives
can be specified.

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

### `--folders`, `--dirs`, `-d`
Only folders are included in the results - files are omitted. When searching for matching `--name` patterns, only folders with a filename (but not path) matching the specified pattern are checked.

### `--help` `-h` `-?`
Print this help text.

### `--hidden` `-a`
Show all hidden files - hidden files are omitted by default.

### `--limit` `-l`
Limit `lspath` to operate non-recursively.

### `--name=<filename>` `-n <filename>`
Only print filepaths containing the given name pattern. Name comparisions
are case-insensitive. If multiple `--name` directives are given, each
filepath need only match one of the names to be printed. Wildcard names
patterns may be used, e.g. `"ABC*.cpp"`.

### `--pipe` `-p`
Each result filepath is printed with a terminating NUL character (ASCII 0) instead of
a newline, which makes the output safe to pipe into commands that accept NUL-separated
input - filepaths containing spaces, quotes, and other special characters are passed
through intact. Filepaths are not escaped or quoted, so `--pipe` cannot be combined
with `--esc`; it also cannot be combined with `--command`. Example:

    lspath -n .rogue --pipe | xargs -0 wc -l

### `--quiet` `-q`
Prevents the `--command` option from displaying each command before executing it.
Does not suppress the execution output. Useful when piping the result of `lspath`
into another command.

### `--relative` `-r`
Prints relative filepaths instead of absolute filepaths.

## Wildcard Patterns
Put patterns in quotes to ensure that LSPath's nonstandard wildcard pattern
'**' is processed correctly. By example:

Pattern         | Description
----------------|-------------------------------------------
`"*"`           |All files in current folder
`"**"`          |All files, recursively
`"**/*.rogue"`  |All .rogue files, recursively
`"A?E"`         |All 3-letter files in current folder starting with `A` and ending with `E`.

# Example

## Show filepaths for all .cpp, and .h files

    lspath "**/*.cpp" "**/*.h"

