# mac-tools

Six single-file command line tools for a Mac that runs Claude Code sessions all day. No dependencies beyond what macOS ships, plus `gh`, `jq` or Ollama where a tool says so. Drop one in your `PATH` and it works.

| Tool | What it does |
|---|---|
| `awake` | Holds a sleep assertion only while Claude sessions justify it |
| `cremote` | Keeps `claude remote-control` servers alive under launchd, so a phone can start sessions on this Mac |
| `ical` | Drives Calendar.app from the shell: list, add, remove, import, clear a date range |
| `mail-unsub` | Unsubscribes from every newsletter Mail.app filed under Promotions or Junk |
| `routines` | Shows every launchd routine on one screen, grouped and colored, and flags the broken ones |
| `veille` | Summarises unread Newsboat articles with Claude, filtered by tag and age |

## Why hold the Mac awake conditionally?

A `caffeinate` you start by hand drains the battery when you walk away. `awake` decides per policy instead: it holds on AC always, on battery above the idle line, and between the floor and that line only while a session transcript was written in the last few minutes.

```sh
awake status   # what it is doing, and why
awake check    # run the policy table once
```

Tunables live in `~/.config/awake`: `AWAKE_IDLE_MIN`, `AWAKE_BATTERY_IDLE`, `AWAKE_BATTERY_FLOOR`, `AWAKE_LID`. With `AWAKE_LID=1` a closed lid on AC stays awake, which needs one sudoers line that `awake help` prints.

## How does the phone reach this Mac?

Each line of `~/.config/cremote/targets` is one server: a name, a directory, optional flags. Every target becomes a launchd agent with `KeepAlive`, so it survives a reboot.

```sh
cremote on | off | restart | status | log
```

`cremote status` prints each environment URL. The Mac still has to be awake, which is what `awake` is for.

## What can ical reach that a calendar API cannot?

Whatever accounts Calendar.app holds, including iCloud, which has no usable public API. Reads go straight to the Calendar SQLite store and are instant. Writes go through AppleScript, addressed by uuid, never by `whose`, which scans the whole calendar and takes minutes.

```sh
ical ls
ical events Perso 2026-09-01 2026-09-30
ical add Perso "Dentiste" "2026-09-14 10:30" -m 30
ical rm Perso <uuid>
```

Dates are read in `$ICAL_TZ`, default `Europe/Paris`, whatever the Mac's own zone. Its built-in help is in French.

## What does mail-unsub actually click?

The RFC 8058 one-click endpoint when the sender offers one, otherwise a `mailto:` sent from the account that received the mail, otherwise the unsubscribe link. One attempt per sender, retried up to three times if mail keeps arriving.

```sh
mail-unsub --dry-run   # list candidates, touch nothing
mail-unsub
```

Candidates are messages carrying a `List-Unsubscribe` header that Mail.app filed under Promotions or Junk. `~/.config/mail-unsub/always` forces a sender in, `never` protects one, both by substring. State lives in `~/.local/state/mail-unsub/state.json`.

## What does routines show?

One line per launchd job: what it does, how often and at what time, its next run and whether it is healthy. It reads three optional plist keys launchd ignores: `Description` (the WHAT column), `Tags` (the groups) and `Frequency` (for a script that gates itself, such as hourly slots that run once a month).

```sh
routines                          # every routine, grouped by tag
routines --tag work --sort next   # one group, soonest first
routines --columns name,what,next # only these columns
routines --problems               # only the broken ones, with evidence; for a daily job
```

It flags a nonzero last exit, a job that never ran on schedule, a stale log and a command missing from the job's own `PATH`. Python 3 standard library only.

## Install

```sh
git clone https://github.com/cyrilghali/mac-tools.git
ln -s "$PWD/mac-tools/awake" ~/.local/bin/awake   # and so on, per tool
```

`awake`, `cremote` and `mail-unsub` are meant to run under launchd. Each prints its own plist guidance with `help` or `--help`.

MIT.
