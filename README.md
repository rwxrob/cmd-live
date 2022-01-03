# Live Streamer Command Line Utility

*Once was called `iam` instead of `live` (`alias iam=live` if needed).*

This bash script helps live streamers communicate the current state of
their stream quickly from the Linux command line by creating and
changing a number of streaming *modes* which are stored in a simple
configuration file. Streamers can also optionally post a corresponding
Twitch chat message, GitHub status, and/or Twitter update. This script
can also be used to post markers in the current live stream to help with
video editing later, lookup user information, ban and timeout users,
list the top channels in a given category, and more.

Each mode has the following properties which can be easily set and get
using the `config` commands:

* Name
* Emoji
* Default Status
* Current Status
* Category
* Tags

Each is added using either the `config.set` or `config.edit` commands.
The `set` command sets it directly from the command line. The `edit`
command opens the configuration `values` file using your configured
default `$EDITOR`.

**Name** is a memorable name uniquely identifying the mode.  Names may
contain dots (.) to separate modes into submodes easily (`playing.dota2`,
`playing.witcher3`, `out.cycling`, `out.watching.birds`,
`out.watching.movie`). This makes for quick and organized tab completion
later.

## Dependencies

Required:

* Bash 4+
* `jq` - JSON Parsing
* `twitch` - Official Twitch Command Line Utility

Optional:

* `gh` - GitHub Status Updates
* `twurl` - Twitter Posts
* `pandoc` - Rich Help Docs

## Security

This script is expected to be installed for a specific user and only
ever run by that user. No additional security vetting for running as an
untrusted user has been done.

## Legal

Copyright 2021 Rob Muhlestein <rob@rwx.gg>  
Released under Apache-2.0 License  
Please mention rwxrob.tv for attribution

## The `about` Command

```
live about
```

Prints Markdown output suitable for pasting into an About text pane.
Also sends to `xclip` if it is detected on the system.

## The `category` Command

```
live category [NEW]
```

Print or change the current mode category (which is still `game_id`
internally in the Twitch API). Prints both the category ID number and
the long written form separated by a space. Note that
changing the category requires the number only and does not immediately
update the current category on Twitch, only the local entry in the
configuration data. Consider adding popularly used categories to the
local configuration. See `list.categories` and `category.select`.

## The `category.select` Command

```
live category.select
```

Change the current mode category (which is still `game_id` internally in
the Twitch API) by selecting from a menu containing all categories
loaded into the local configuration cache values file (see `config path`
for more).  Note that this does not update the current category on
  Twitch, only the local entry in the configuration data.  See
  `list.categories` and `category`.

## The `config` Command

```
live config
live config KEY
live config.set KEY VALUE
live config.set KEY ""
live config.keys
live config.values
live config.directory
live config.path [file]
live config.edit [file]
live config.delete
live config.read
live config.write
live config.dump
```

The `config` command is for reading, writing, and displaying standard
open desktop configuration properties. 

### Arguments

With no arguments calls `dump` and outputs all the currently cached
configuration settings.

With a single KEY argument fetches the value for that key and outputs
it unless it is one of the following special (reserved) key names:

* `directory` full path to config directory
* `path`   full path to specific config file (default: `values`) 
* `edit`   opens config file in editor (default: `editor` or `$EDITOR)
* `keys`   output the configuration keys, one per line
* `values` output the configuration values, one per line
* `delete` if key argument then delete a specific key, otherwise prompt
* `read` reads the configuration file into CONF associative array
* `write` write the CONF associative array to the configuration file
* `dump` write the flattened CONF associative array to standard output

With more than one argument the remaining arguments after the KEY will
be combined into the VALUE and written to a `values` file in the
configuration directory. 

### Configuration Directory

The configuration directory path relies on the following environment
variables:

* `EXE` - defaults to name of currently running command (live)
* `HOME` - checked for `$HOME/.config/$EXE/values`
* `XDG_CONFIG_HOME` - overrides `$HOME/.config`
* `CONFIG_DIR` - full path to directory containing `values` file

The `CONFIG_DIR` always takes priority over anything else if set, but is
never implied. If the directory does not exist it will be created the
first time a value is set.

### Configuration `values` File Format

The file (which is almost always located at
`~/.config/live/values`) uses the simplest possible format to
facilitate standard UNIX parsing and filtering with any number of
existing tools (and no `jq` dependency).

* One KEY=VALUE per line
* KEYs may be anything but the equal sign (`=`)
* VALUEs may be anything but line returns must be escaped

Note that, although similar, this is *not* the same as Java properties
and other similar format. It is designed for ultimate simplicity,
efficiency, and portability.

## The `create` Command

```
live create MODE
```

Create a new mode with interactive prompt. Note that modes can also be
created directly by simply adding the lines to the `live config
path` values file.

## The `delete` Command

```
live delete MODE
```

Delete all data for a specific. WARNING: Does not ask for confirmation.

## The `description` Command

```
live description [NEW]
```

Print or change the current mode description.

## The `emoji` Command

```
live emoji [NEW]
```

Print or change the current mode emoji.

## The `emojis` Command

```
live list.emojis
live emojis
```

List emojis from all modes. See `emoji`.

## The `help` Command

```
live help [COMMAND]
```

Displays specific help information. If no argument is passed displays
general help information (main). Otherwise, the documentation for the
specific argument keyword is displayed, which usually corresponds to
a COMMAND name (but not necessarily). All documentation is written in
GitHub Flavored Markdown and will displayed as a web page if `pandoc`
and `$HELP_BROWSER` are detected, otherwise, just the Markdown is sent
to `$PAGER` (default: more).

Also see `readme` and `usage` commands.

## The `list.categories` Command

```
live list.categories
```

List available categories.

## The `list.emojis` Command

```
live list.emojis
live emojis
```

List emojis from all modes. See `emoji`.

## The `list.modes` Command

```
live list.modes
live modes
```

List available modes.

## The `list.tags` Command

```
live list.tags
```

List available tags. Note that tags (without list) is an entirely
different command. To add tags to the local configuration cache edit the
configuration file directly with `config edit` each tag should be in the
format of the following example:

```
tag.cozy=adc4a830-07f5-457b-95e5-5ab6cc1f9af3
```

Note that there cannot be any whitespace anywhere in the line.

## The `list.titles` Command

```
live list.titles
live titles
```

List all titles in alphabetical order by mode name.

## The `list.titles.modes` Command

```
live list.titles.modes
```

Same as list.titles but also show the mode name.

## The `mode` Command

```
live mode [NEW]
```

Print the current mode name (from local cache) or set to a new one and
update it on Twitch as well. Note that this does not fetch the current
remote Twitch status so it is possible that if Twitch stream info and
tags have been set some other way that the local cache will be out of
sync. A quick check of the current title should help identify when this
happens, but it is inconsequential since usually a new mode is wanted
anyway.

## The `modes` Command

```
live list.modes
live modes
```

List available modes.

## Generate `README.md` File

```
live readme > README.md
```

The `readme` command will output the embedded help documentation in raw
GitHub Flavored Markdown suitable for use as a `README.md` file on
GitHub or similar hosting service.

## The `status` Command

```
live status [NEW]
```

Print or change the current status. Note this does not change the
status.default. To remove and return to status.default use
status.clear.

## The `status.clear` Command

```
live status.clear
```

Clears the current status (effectively switching back to status.default)
.

## The `status.default` Command

```
live status.default [NEW]
```

Print or change the current mode status default. (See status also.)

## The `tag` Command

```
live tag (UUID|NAME)
```

Lookup a tag in the local configuration cache by UUID or short name
(NAME). See list.tags for how tags are added to the cache.

## The `tags` Command

```
live tags [NEW]
```

Print or change the current Twitch tags list. Twitch limits tags to five
total. Tags with their Twitch unique identifiers must first be added
before they will work. See `list.tags`.

## The `title` Command

```
live title [MODE]
```

Print the current emoji and status suitable for setting twitch.title.
The current status is used if MODE is not passed. This is default
command when no arguments are passed.

## The `titles` Command

```
live list.titles
live titles
```

List all titles in alphabetical order by mode name.

## The `twitch.category` Command

```
live twitch.category [NEW]
```

Set or get the remote Twitch category (game_id) for the current
`twitch.id`. The special SAME category will skip any attempt to update
the current Twitch category. This is useful for break modes and such.

## The `twitch.channel` Command

```
live twitch.channel
live channel
```

Print the Twitch API channel JSON data for the current `twitch.id`.

## The `twitch.title` Command

```
live twitch.title [NEW]
```

Set or get the remote Twitch title for the current `twitch.id`.

## The `twitch.user` Command

```
live twitch.user [NAME]
```

Fetch and print summary for a give Twitch user by name. If no name is
provided, will display summary for the current `twitch.name`.

## The `usage` Command

```
live usage
```

Display all possible commands. Note that this is usually easier by
simply using tab completion instead.

----

*Autogenerated Mon Jan  3 09:25:26 AM EST 2022*

