# panicparse

Parses panic stack traces, densifies and deduplicates goroutines with similar
stack traces. Helps debugging crashes and deadlocks in heavily parallelized
process.

[![GoDoc](https://godoc.org/github.com/maruel/panicparse/stack?status.svg)](https://godoc.org/github.com/maruel/panicparse/stack)
[![Go Report Card](https://goreportcard.com/badge/github.com/maruel/panicparse)](https://goreportcard.com/report/github.com/maruel/panicparse)
[![Coverage Status](https://codecov.io/gh/maruel/panicparse/graph/badge.svg)](https://codecov.io/gh/maruel/panicparse)
[![Build Status](https://travis-ci.org/maruel/panicparse.svg)](https://travis-ci.org/maruel/panicparse)
[![Fuzzit Status](https://app.fuzzit.dev/badge?org_id=maruel)](https://app.fuzzit.dev/orgs/maruel/dashboard)


panicparse helps make sense of Go crash dumps:

![Screencast](https://raw.githubusercontent.com/wiki/maruel/panicparse/parse.gif "Screencast")


## Features

   * &gt;50% more compact output than original stack dump yet more readable.
   * Exported symbols are bold, private symbols are darker.
   * Stdlib is green, main is yellow, rest is red.
   * Deduplicates redundant goroutine stacks. Useful for large server crashes.
   * Arguments as pointer IDs instead of raw pointer values.
   * Pushes stdlib-only stacks at the bottom to help focus on important code.
   * Usable as a library!
     [![GoDoc](https://godoc.org/github.com/maruel/panicparse/stack?status.svg)](https://godoc.org/github.com/maruel/panicparse/stack)
     * Warning: please pin the major version (i.e. vendor it via
       [dep](https://github.com/golang/dep)) as breaking changes happen on
       [major version update](https://semver.org/).
   * Parses the source files if available to augment the output.
   * Works on Windows.


## Authors

`panicparse` was created with ❤️️ and passion by [Marc-Antoine
Ruel](https://github.com/maruel) and
[friends](https://github.com/maruel/panicparse/graphs/contributors).


## Installation

    go get github.com/maruel/panicparse/cmd/pp


## Usage

### Piping a stack trace from another process

#### TL;DR

   * Ubuntu (bash v4 or zsh): `|&`
   * OSX, [install bash 4+](README.md#updating-bash-on-osx), then: `|&`
   * Windows _or_ OSX with stock bash v3: `2>&1 |`
   * [Fish](http://fishshell.com/) shell: `^|`


#### Longer version

`pp` streams its stdin to stdout as long as it doesn't detect any panic.
`panic()` and Go's native deadlock detector [print to
stderr](https://golang.org/src/runtime/panic1.go) via the native [`print()`
function](https://golang.org/pkg/builtin/#print).


**Bash v4** or **zsh**: `|&` tells the shell to redirect stderr to stdout,
it's an alias for `2>&1 |` ([bash
v4](https://www.gnu.org/software/bash/manual/bash.html#Pipelines),
[zsh](http://zsh.sourceforge.net/Doc/Release/Shell-Grammar.html#Simple-Commands-_0026-Pipelines)):

    go test -v |&pp


**Windows or OSX native bash** [(which is
3.2.57)](http://meta.ath0.com/2012/02/05/apples-great-gpl-purge/): They don't
have this shortcut, so use the long form:

    go test -v 2>&1 | pp


**Fish**: It uses [^ for stderr
redirection](http://fishshell.com/docs/current/tutorial.html#tut_pipes_and_redirections)
so the shortcut is `^|`:

    go test -v ^|pp


**PowerShell**: [It has broken `2>&1` redirection](https://connect.microsoft.com/PowerShell/feedback/details/765551/in-powershell-v3-you-cant-redirect-stderr-to-stdout-without-generating-error-records). The workaround is to shell out to cmd.exe. :(


### Investigate deadlock

On POSIX, use `Ctrl-\` to send SIGQUIT to your process, `pp` will ignore
the signal and will parse the stack trace.


### Parsing from a file

To dump to a file then parse, pass the file path of a stack trace

    go test 2> stack.txt
    pp stack.txt


## Tips

### Disable inlining

Starting with go1.11, the toolchain starts to inline more often. This causes
traces to be less informative. You can use the following to help diagnosing
issues:

    go install -gcflags '-N -l' path/to/foo
    foo |& pp

or

    go test -gcflags '-N -l' ./... |& pp


### GOTRACEBACK

Starting with Go 1.6, [`GOTRACEBACK`](https://golang.org/pkg/runtime/) defaults
to `single` instead of `all` / `1` that was used in 1.5 and before. To get all
goroutines trace and not just the crashing one, set the environment variable:

    export GOTRACEBACK=all

or `set GOTRACEBACK=all` on Windows. Probably worth to put it in your `.bashrc`.


### Updating bash on OSX

Install bash v4+ on OSX via [homebrew](http://brew.sh) or
[macports](https://www.macports.org/). Your future self will appreciate having
done that.


### If you have `/usr/bin/pp` installed

If you try `pp` for the first time and you get:

    Creating tables and indexes...
    Done.

and/or

    /usr/bin/pp5.18: No input files specified

you may be running the _Perl PAR Packager_ instead of panicparse.

You have two choices, either you put `$GOPATH/bin` at the begining of `$PATH` or
use long name `panicparse` with:

    go get github.com/maruel/panicparse

then using `panicparse` instead of `pp`:

    go test 2> panicparse
