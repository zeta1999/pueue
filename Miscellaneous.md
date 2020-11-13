- [Aliases](#aliases)
- [Callbacks](#callbacks)
- [Shell completion files](#shell-completion-files)
- [Scripting](#scripting)
    * [JSON support](#json-support)
- [Logs](#logs)

## Aliases

To get basic aliasing, simply put a `pueue_aliases.yml` besides your `pueue.yml`.
Its contents should look something like this:

```yaml
ls: 'ls -ahl'
rsync: 'rsync --recursive --partial --perms --progress'
```

When adding a command to pueue, the **first** word will then be checked for the alias.
This means, that for instance `ls ~/ && ls /` will result in `ls -ahl ~/ && ls /`.\
If you want multiple aliases in a single task, it's probably best to either create a task for each command or to write a custom script.

## Callbacks

You can specify a callback that will be called every time a task finishes.
The callback can be parameterized in the `pueue.yml` file in the `daemon` section with some variables.

These are the available variables that can be used to create a command:

- `{{ id }}`
- `{{ command }}`
- `{{ path }}`
- `{{ result }}` (Success, Killed, etc.)
- `{{ exit_code }}` The shell exit code (0-255, can be `None` for killed/unspawned processes)
- `{{ group }}`
- `{{ start }}` (times are UNIX timestamps or empty)
- `{{ end }}`
- `{{ enqueue }}`

Example callback:

```yaml
callback: "notify-send \"Task {{ id }}\nCommand: {{ command }}\nPath: {{ path }}\nFinished with status '{{ result }}'\nTook: $(bc <<< \"{{end}} - {{start}}\") seconds\""
```

Here's another example for macOS notifications:

```yaml
callback: "osascript -e 'display notification \"Finished with status {{ result }}\" with title \"Pueue task {{ id }}\" subtitle \"{{ command }}\"'"
```

Depending on how complex your callback becomes it might make sense to put into an external file:

```yaml
callback: '/path/to/callback "{{id}}" "{{command}}" "{{path}}" "{{result}}" "{{group}}" "{{start}}" "{{end}}" "{{enqueue}}"'
```

Make sure to restart the daemon after you make changes to `pueue.yml`.

## Shell completion files

Shell completion files can be created on the fly with `pueue completions $shell $directory`.
There's also a `build_completions.sh` script, which creates all completion files in the `utils/completions` directory.

Here's an example of where those files should be placed in Arch-Linux:

- Zsh: `/usr/share/zsh/site-functions/_pueue`
- Bash: `/usr/share/bash-completion/completions/pueue.bash`
- Fish: `/usr/share/fish/completions/pueue.fish`

The paths might differ for your Linux distribution, so please double-check those paths.

## Scripting

When calling pueue commands in a script, you might need to sleep for a short amount of time for now.
The pueue server processes requests asynchronously, whilst the TaskManager runs it's own update loop with a small sleep.
(The TaskManager handles everything related to starting, stopping and communicating with processes.)

A sleep in scripts will probably become irrelevant, as soon as this bug in rust-lang is fixed: [issue](https://github.com/rust-lang/rust/issues/39364)

### JSON Support

The Pueue client `status` and `log` commands support JSON output with the `-j` flag.
This can also be used to easily incorporate it into window manager bars, such as i3bar.

## Logs

All logs can be found in `${pueue_directory}/logs`.
Logs of previous Pueue sessions will be created whenever you issue a `reset` or `clean`.
In case the daemon fails or something goes wrong, the daemon will print to `stdout`/`stderr`.
If the daemon crashes or something goes wrong, please set the debug level to `-vvvv` and create an issue with the log!

If you want to dig right into it, you can compile and run it yourself with a debug build.
This would help me a lot!