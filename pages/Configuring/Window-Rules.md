# Table of contents

{{< toc >}}

# Window Rules V1

You can set window rules to achieve different behaviours from the active container.

## Syntax

```ini
windowrule=RULE,WINDOW
```

- `RULE` is a [rule](#rules) (and a param if applicable)
- `WINDOW` is a RegEx, either:
  - plain RegEx (for matching a window class);
  - `title:` followed by a regex (for matching a window's title)

### Examples

```ini
windowrule=float,^(kitty)$
windowrule=move 0 0,title:^(Firefox)(.*)$
```

# Window Rules V2

In order to allow more flexible rules, while retaining compatibility with the above
rule system, window rules V2 were implemented.

In V2, you are allowed to match multiple variables.

the `RULE` field is unchanged, but in the `WINDOW` field, you can put regexes
for multiple values like so:

```ini
windowrulev2 = float,class:^(kitty)$,title:^(kitty)$
```

For now, the supported fields are:

```ini
class - class regex
title - title regex
xwayland - 0/1
floating - 0/1
fullscreen - 0/1
pinned - 0/1
```

Keep in mind that you _have_ to declare at least one field, but not all.

{{< hint type=tip >}}

To get more information about a window's class, title, XWayland status or its size,
you can use `hyprctl clients`.

{{< /hint >}}

## Rules

| Rule | Description | Dynamic |
| ---- | ----------- | ----------- |
| float | floats a window | |
| tile | tiles a window | |
| fullscreen | fullscreens a window | |
| nofullscreenrequest | prevents windows from requesting fullscreen mode, you can still manually toggle fullscreen. | |
| move \[x\] \[y\] | moves a floating window (x,y -> int or %, e.g. 20% or 100. You are also allowed to do `100%-` for the right/bottom anchor, e.g. `100%-20`) Additionally, you can also do `cursor [x] [y]` where x and y are either pixels or percent. Percent is calculated from the window's size. | |
| size \[x\] \[y\] | resizes a floating window (x,y -> int or %, e.g. 20% or 100) | |
| minsize \[x\] \[y\] | sets the minimum size on creation (x,y -> int) | |
| maxsize \[x\] \[y\] | sets the maximum size on creation (x,y -> int) | |
| center | if the window is floating, will center it on the monitor | |
| pseudo | pseudotiles a window | |
| monitor \[id\] | sets the monitor on which a window should open | |
| workspace \[w\] | sets the workspace on which a window should open (for workspace syntax, see [dispatchers->workspaces](../Dispatchers#workspaces)). You can also make \[w\] to `unset`, will unset all previous workspace rules applied to this window. You can also add `silent` after the workspace to make the window open silently. | |
| opacity \[a\] | additional opacity multiplier. Options for a: `float` -> sets an opacity OR `float float` -> sets activeopacity and inactiveopacity respectively |&check;|
| opaque | forces the window to be opaque (can be toggled with the toggleopaque dispatcher) |&check;|
| animation \[style\] (\[opt\]) | forces an animation onto a window, with a selected opt. Opt is optional. |&check;|
| rounding \[x\] | forces the application to have X pixels of rounding, ignoring the set default (in `decoration:rounding`). Has to be an int. |&check;|
| noblur | disables blur for the window |&check;|
| nofocus | disables focus to the window | |
| noborder | disables borders for the window |&check;|
| noshadow | disables shadows for the window |&check;|
| forceinput | forces an XWayland window to receive input, even if it requests not to do so. (Might fix issues like e.g. Game Launchers not receiving focus for some reason) | |
| windowdance | forces an XWayland window to never refocus, used for games/applications like Rhythm Doctor | |
| pin | pins the window *note: floating only* | |
| noanim | disables the animations for the window |&check;|
| bordercolor \[c\] | force the bordercolor of the window. Options for c: `color` -> sets the active border color OR `color color` -> sets the active and inactive border color of the window. See [variables->colors](../Variables#variable_types) for color definition. |&check;|
| idleinhibit \[mode\] | sets an idle inhibit rule for the window. If active, apps like `swayidle` will not fire. Modes: `none`, `always`, `focus`, `fullscreen` | |
| unset | removes all previously set rules for the given parameters. Please note it has to match EXACTLY. | |
| nomaxsize | removes max size limitations. Especially useful with windows that report invalid max sizes (e.g. winecfg) | |

### Example Rules

```ini
windowrule = move 100 100,^(kitty)$ # moves kitty to 100 100
windowrule = animation popin,^(kitty)$ # sets the animation style for kitty
windowrule = noblur,^(firefox)$ # disables blur for firefox
windowrule = move cursor -50% -50%,^(kitty)$ # moves kitty to the center of the cursor
windowrulev2 = bordercolor rgb(FF0000) rgb(880808),fullscreen:1 # set bordercolor to red if window is fullscreen
windowrulev2 = bordercolor rgb(FFFF00),title:^(.*Hyprland.*)$ #set bordercolor to yellow when title contains Hyprland
```

### Notes

Rules that are marked as _Dynamic_ will be reevaluated if the matching property of the window changes. For instance, if a rule is defined that changes the bordercolor of a window when it is floating, then the bordercolor will change to the requested color when it is set to floating, and revert to the default color when it is tiled again. Dynamic matching properties are _title_, _floating_, _fullscreen_ and _pinned_.

Rules will be processed from top to bottom, where the _last_ match will take precedence.
i.e.

```ini
windowrulev2 = opacity 0.8 0.8,class:^(kitty)$
windowrulev2 = opacity 0.5 0.5,floating:1
```

-> all kitty windows will have opacity 0.8, except if they are floating. Then they will have opacity 0.5.
-> all floating windows will have opacity 0.5.

```ini
windowrulev2 = opacity 0.5 0.5,floating:1
windowrulev2 = opacity 0.8 0.8,class:^(kitty)$
```

-> all kitty windows will have opacity 0.8, also if they are floating.
-> all other floating windows will have opacity 0.5.


{{< hint type=tip >}}

Opacity is _always_ a PRODUCT of all opacities. E.g. `active_opacity` to
0.5 and windowrule opacity to 0.5 will result in a total opacity 0.25. You are
allowed to set opacities over 1, but any opacity product over 1 will cause
graphical glitches. E.g. `0.5 * 2 = 1`, and it will be fine, `0.5 * 4` will cause
graphical glitches.
{{< /hint >}}
