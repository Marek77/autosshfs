# autosshfs
A command-line tool using `~/.ssh/config` to automatically connect sshfs mountpoints. This may be useful for people who have a lot of Hosts in their `~/.ssh/config` and want hassle-free sshfs mounts.

## How does it work
`autosshfs` parses the `ssh` configuration file and allows to create an (almost) automatic sshfs filesystem. While `autosshfs` supports its own configuration directives in `~.ssh/config`, no configuration directives are obligatory.

## Dependencies

`autosshfs` has very few dependencies. Install the following packages (Fedora / RHEL):

- fuse-sshfs
- perl-File-Path
- perl-Getopt-Long
- perl-IO-Interactive

Put `autosshfs` into `$HOME/bin` or `/usr/local/bin` and you're good to go.

## `autosshfs` usage

```
autosshfs [--file <file>] [--connect [<target>[,<target2>,...]]] [--disconnect [<target>[,<target2>,...]]] [--reconnect [<target>[,<target2>,...]]] [--list] [--mounts] [--verbose] [--help]

    -h          : Display this help message and exit
    -f <file>   : Use the specified file instead of $HOME/.ssh/config
    -l          : List targets and exit
    -m          : List mount status and exit
    -c          : Connect all targets
    -c <target> : Connect a specific target or a comma-separated list of targets
    -d          : Disconnect all targets
    -d <target> : Disconnect a specific target or a comma-separated list of targets
    -r          : Reconnect all targets
    -r <target> : Reconnect a specific target or a comma-separated list of targets
    -v          : Verbose; also implies '-m' after connect/disconnect/reconnect

```

## Supported directives

The `autosshfs` directives "hide" in the comments of the `ssh` configuration file, one per line. The line containing an `autosshfs` directive must start with `#!!!#`, and may be preceded by whitespace. The format is:

`#!!!# <variable> <value>`

All the directives are case-insensitive.

### `Host *` section
These directives are ignored if they aren't in the `Host *` section.

- `BaseDir`: the "root" of the sshfs "tree". Default: `$HOME/autosshfs`
- `PromptForAll`: if `autosshfs` should ask for user's confirmation for the actions concerning 'all' targets (`--connect`, `--disconnect`, `--reconnect` without arguments). Possible values: `yes`, `no`. If set to `yes` and `autosshfs` runs in a non-interactive environment (without a tty), it will exit without performing the action. Default: `no`

### `Host <name>` section
These directives are ignored if they aren't in the `Host <name>` section.

- `RemoteDir`: remote mountpoint to connect. Default: `$HOME` of the `$USER` used to connect to the remote server. 
- `LocalDir`: local mountpoint (it will be created if it doesn't exist and the user running `autosshfs` has appropriate rights), relative to `BaseDir`. Default: `$BaseDir/<name>`
- `ExcludeFromAll`: `autosshfs` allows to mount / unmount directories for all the `Host <name>` entries in the configuration file. This directive removes the corresponding `Host` from this list.

## Tips

To mount an `~/.ssh/config` entry automatically upon connection, put this into `~/.ssh/config` (let's use `Host somemachine`):
```
...

Host somemachine
    LocalCommand autosshfs -c %n
...

Host *
    PermitLocalCommand yes
```
See `man ssh_config`.

It should be possible to [hack](http://serverfault.com/a/641004) something similar to automatically unmount the `sshfs` mountpoint upon the ssh disconnect (ssh doesn't have a directive corresponding to `LocalCommand` to be run after disconnect).

To disconnect all the `sshfs` mountpoints, execute: `autosshfs -d`. 
**NOTE:** as usual, this may fail if the mountpoint is in use; see `man fuser`.

## TODO
- man page (based on this README)
- syntax completion
- Makefile
- RPM packaging
- ???
