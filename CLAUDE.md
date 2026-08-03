# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

LSPath is a cross-platform (macOS/Linux/Windows) command-line tool that lists full filepaths matching wildcard search patterns, with optional command execution on each result (`--command` with `$`/`$$` placeholder substitution). It is written in [Rogue](https://github.com/brombres/Rogue), which compiles Rogue → C → native executable, and is built with Rogo (Rogue's build tool). A local copy of the Rogue repo is at `~/Projects/Rogue` — consult it for language syntax and standard library sources (e.g. `File`, `FilePattern`, `CommandLineParser`).

## Build Commands

Requires Rogue/Rogo installed (github.com/brombres/Rogue). Run from the repo root:

- `rogo` — build (if needed) and run
- `rogo build` — incremental build; only recompiles Rogue → C → exe when sources are newer
- `rogo rebuild` — force a full rebuild (`rogo rebuild debug` / `rogo rebuild release` for a specific mode; release is the default)
- `rogo run` — run the built executable (`Build/LSPath-<OS>`, e.g. `Build/LSPath-macOS`)
- `rogo clean` — delete `Build/` and `.rogo/`
- `rogo install` — build and install a `lspath` launcher (to `/usr/local/bin` on Unix)
- `rogo help` — list all build commands

There is no test suite; verify changes by building and running `lspath` against real file patterns.

## Release Workflow

Version and date are defined in two `$define` lines at the top of `Source/LSPath.rogue` and mirrored in the README's About table. Do not edit them by hand — Rogo automates this:

- `rogo update_version <x.y>` — rewrites VERSION/DATE in source and README
- `rogo commit <x.y>` — update_version + `git commit -am "[vx.y]"`
- `rogo publish <x.y>` — commit, push, merge current branch into main (interactive), and create a GitHub release via `gh`

Day-to-day work happens on the `develop` branch; `main` is the release branch.

## Architecture

The entire tool is a single class in `Source/LSPath.rogue` (~360 lines). Flow in `init()`:

1. Parse options with `Console/CommandLineParser` (each `--option` has a short alias; `&multi` options like `--name`/`--exclude`/`--grep` accumulate into lists).
2. Expand filepath args: bare folders become `folder/*` (with `--limit`) or `folder/**` patterns; non-existent paths are treated as wildcard patterns and expanded via `File.listing`.
3. `add()` filters each candidate (hidden files, `--files`/`--folders`, `--exclude` then `--name` FilePattern matching, then `--grep` content matching) and stores absolute paths in an ordered `[String:String]` map for deduplication.
4. Output: either print filepaths (optionally `--relative`, `--esc`), or run `--command`, substituting `$`-placeholders per file (command runs N times) or `$$`-placeholders (command runs once with the whole list; `$$[N]` indexes into it).

Other files:

- `Build.rogue` — Rogo build script; project settings (compiler flags, launcher name) are in the `augment Build` block at the top and can be overridden with a `Local.settings` file.
- `Morlock/lspath.rogue` — package definition for installation via the Morlock package manager (`morlock install brombres/lspath`).
- `Build/` and `.rogo/` — generated build output (committed, including per-OS transpiled C); don't hand-edit.

## Workflow

Auto-commit is the default in this repo: after completing a change, commit it without waiting to be asked (on `develop`, following the existing `[topic] Description` commit message style). Version-number releases still go through `rogo commit` / `rogo publish`.

## Documentation Sync

Option documentation lives in three places that must be kept in sync when adding or changing options: the `print_usage` help text in `Source/LSPath.rogue`, the parser `option(...)` list in `init()`, and `README.md`.
