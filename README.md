# Plugin Debugger

A MUSHclient plugin (`Plugin_Debugger.xml`, name `Plugin_Debugger`, command `dbg`) that answers questions like "what keeps sending `stop`?" and "which plugin is hammering another one?" without touching any of your other plugins.

It reads MUSHclient's own Trace output and prints a cleaned-up version in the world window: what fired, who called it, and how often. Whenever a command goes to the game, it shows the command together with the trigger or alias matches and the last game line that came right before it.

## Install
1. Put `Plugin_Debugger.xml` in your plugins folder and load it (File > Plugins > Add).
2. Type `dbg on`. This switches MUSHclient's trace on for you; `dbg off` switches it off again.
3. Type `dbg window` to choose what to watch, or `dbg help` for the commands.

The options window uses the themed window library that comes with the Aardwolf MUSHclient package; without it, everything still works from the commands.

## What you see
Every line starts with `[dbg]` and a colored tag:

| Tag | Meaning |
|---|---|
| `[trigger]` `[alias]` `[timer]` | a match or firing, shown with the script it ran: `Plugin: trigger "pattern"  ->  script("the line")` |
| `[call]` | one plugin called a function in another with CallPlugin: `caller -> plugin: function(arguments)` |
| `[callback]` | MUSHclient ran a plugin function itself (`OnPluginLineReceived`, `OnPluginSent`, ...) |
| `[broadcast]` | a plugin announced something to all the others (GMCP data, `dock_ready`, ...) |
| `[script]` | a script function whose trigger, alias or timer line was not seen |
| `[sent]` | a command sent to the game: `SENT "stop"`, with `[before]` lines under it showing the matches that led to it and the last line the game sent |
| `[other]` | anything else |

Repeated lines in a burst are collapsed into `x12`. Window-drawing calls (`WindowText`, `WindowRectOp`, ...) are hidden by default because they drown everything else.

## The options window (`dbg window`)
Two columns, a checkbox on every row:

- **Plugins** (left): every loaded plugin, plus `(world)` for your world file's own triggers, aliases and scripts. Stock Aardwolf plugins are marked.
- **Kinds of line** (right): the tags from the table above.
- **All** and **None** buttons sit on top of each column. Press **None**, then tick the few plugins you care about, to watch just those.
- The button at the top left turns the debugger on and off. The line next to it says how much is being watched.
- If `dbg ignore` rules are active, the footer says so, since they hide lines even when the box is ticked.

A call counts for a plugin if either end of it is ticked. Sent commands are controlled only by the `[sent]` box, so unticking plugins never hides a `SENT` line. Choices are saved. Run `dbg window` again to hide the window.

## Commands
| | |
|---|---|
| `dbg on` / `dbg off` | start or stop |
| `dbg window` | the options window |
| `dbg options` | the current settings |
| `dbg watch triggers aliases` | show only those kinds (plural works) |
| `dbg watch MyPlugin` | show only lines that mention that text, usually a plugin name |
| `dbg watch stock` / `dbg watch thirdparty` | only the plugins that come with the Aardwolf package, or only the others. `dbg stock` lists which ones count as stock |
| `dbg watch clear` | show everything again (also `dbg watch all`) |
| `dbg ignore calls callbacks` | hide whole kinds |
| `dbg ignore stock` | hide a whole group |
| `dbg ignore gmcpdata` | hide any line containing that text |
| `dbg unwatch WORD` / `dbg unignore WORD` / `dbg ignore clear` | take rules back out |
| `dbg noise windowrectop` | let one kind of hidden window-drawing line through (`dbg noise all`, `dbg noise off`; not remembered) |
| `dbg summary [seconds]` | who called what, and how often, over the last 10 seconds by default. Finds runaway calls |
| `dbg last [number]` | the last events, including ones the live view did not show (default 30) |
| `dbg log on` / `dbg log off` | write what is watched to `plugin_debugger.log` in the logs folder (5 MB, then it rolls over to `plugin_debugger.old.log`) |
| `dbg quiet on` / `dbg quiet off` | keep the trace lines out of MUSHclient's Trace window while this runs |

The watch and ignore rules apply to the screen and to the log alike. The window and the commands edit the same settings: after `dbg watch stock`, the window shows the stock plugins ticked.

## Example: "I never typed stop"
```
dbg on
dbg watch triggers aliases sent
```
When the stray command goes out, you get something like:
```
[dbg] [sent] SENT "stop"
[dbg] [before] Combat: trigger "^Your target flees!$"  ->  flee_stop("Your target flees!")
[dbg] [before] last line from the game: "Your target flees!"
```

## If nothing shows up
`dbg options` shows **Trace lines seen**. If it stays at 0 while the debugger is on, MUSHclient is not passing trace output to plugins, or another plugin is taking it first. The plugin's text is matched on the wording MUSHclient uses for calls, callbacks, and trigger, alias and timer matches; lines it does not recognize still appear as `[other]`, so a line in an unexpected format is shown rather than lost. If you find one that should have been recognized, send me a sample.
