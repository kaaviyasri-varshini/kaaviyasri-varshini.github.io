Title: The Lazy Typist's Guide to Shell History Expansion
Date: 2026-09-09
Category: Terminal
Tags: bash, zsh, shell, terminal, history, productivity, cli, tips
Slug: the-lazy-typists-guide-to-shell-history-expansion

If you've spent any time in a terminal, you've probably typed a command, hit enter, watched it fail with "Permission denied," and then sighed as you retyped the whole thing with `sudo` stuck on the front. There's a better way, and it's been sitting in your shell this whole time.

## The Classic: `sudo !!`

`!!` is bash and zsh's shorthand for "the last command I ran." Prefix it with `sudo`, and your shell expands it into the previous command with elevated privileges:

```bash
$ apt update
Reading package lists... Done
E: Could not open lock file /var/lib/dpkg/lock-frontend - Permission denied

$ sudo !!
sudo apt update
```

Most shells will echo the expanded command before running it, so there's no mystery about what's actually executing. It's a small trick, but once it's muscle memory, retyping a failed command feels almost old-fashioned.

The catch: `!!` re-runs the command *exactly* as you typed it, typos included. If you fat-fingered a flag or misspelled a package name, `sudo !!` will happily repeat the mistake with more permissions.

## It Doesn't Stop at `sudo`

`!!` is just one piece of a much larger event-designator system that bash and zsh inherited from the original C shell. A few of the more useful ones:

- **`!$`** — the *last argument* of the previous command. Handy when you `mkdir some/long/nested/path` and immediately want to `cd !$` into it.
- **`!^`** — the *first argument* of the previous command.
- **`!*`** — *all arguments* of the previous command, useful for reusing a whole file list.
- **`!n`** — re-run command number `n` from your history (check `history` for the number).
- **`!string`** — re-run the most recent command starting with `string`. `!git` reruns your last `git`-something.
- **`!?string?`** — re-run the most recent command containing `string` anywhere.
- **`^old^new`** — quick substitution: reruns the previous command with `old` replaced by `new`. Great for fixing a single typo without retyping everything.

A quick example combining a couple of these:

```bash
$ touch report_final_v2.md
$ mv !$ report.md
mv report_final_v2.md report.md
```

## The zsh Extras

zsh supports all of the `!`-expansions above natively — they're a Bourne/C-shell holdover, not bash-specific. But zsh also layers on its own history search that's worth knowing:

- Type a few characters and press **Ctrl+R** for reverse incremental search, same as bash.
- Type the start of a command and press the **Up arrow** — with the standard history bindings, this searches history for lines *starting with* what you've typed, not just cycling chronologically.
- The `sudo` Oh My Zsh plugin (if enabled) binds **Esc twice** to prepend `sudo` to the current line — a nice complement to `sudo !!`, since it works even before you've run the command.

## Why Bother

None of this is essential — you can always arrow-up and edit, or just retype. But history expansion shaves down the friction of the "run, fail, fix, rerun" loop that makes up a surprising share of a terminal session.

`sudo !!` is the gateway drug. `!$`, `!*`, and `^old^new` are where it starts actually changing how you work.
