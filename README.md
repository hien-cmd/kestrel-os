# Kestrel OS

A small desktop environment that lives entirely in one HTML file. No server, no framework, no build step. Open the file in a browser and you get a window manager, a working shell, a code editor with live preview, a tabbed web browser, a persistent filesystem and a handful of apps, all written in plain JavaScript.

Version 2.0.

## Running it

1. Save the file as `kestrel.html` (any name works).
2. Open it in a modern browser (Chrome, Edge, Firefox or Safari).

That's it. The only external request is for two web fonts (Space Grotesk and JetBrains Mono) from Google Fonts; without a connection it falls back to system fonts.

On first boot you'll see a short BIOS style boot log, then a Terminal and a Files window open with a welcome notification.

## What's inside

| App | What it does |
| --- | --- |
| **Terminal** | `ksh`, a real little shell with pipes, redirects, variables, loops and scripts |
| **Files** | Browse, create, rename, duplicate and delete files and folders |
| **Code** | Editor with a file tree, tabs, syntax highlighting, find and replace, run and live preview |
| **Browser** | Tabs, history, bookmarks; shows real web pages, local files and built in `kestrel://` pages |
| **TextEdit** | Plain text notes with save, save as and unsaved change warnings |
| **Paint** | Draw with a palette and brush sizes, save PNGs to Pictures |
| **Studio** | 16 step sequencer with synthesised drums, bass and lead (Web Audio, no samples) |
| **Snake** | Classic snake with speed ups and a saved high score |
| **Calculator** | Keyboard friendly arithmetic |
| **Settings** | Theme, wallpaper, accent colour, animation, user name, factory reset |
| **About** | What this machine is |

## The desktop

* Drag a title bar to move a window. Drag it to the **top edge** to maximise, or to the **left or right edge** to snap it to half the screen.
* Double click a title bar to maximise or restore.
* The three lamps are minimise (amber), maximise (green) and close (red).
* Resize from any edge or the bottom right corner.
* Right click the desktop for a menu (new terminal, wallpaper, theme, settings).
* Double click a desktop icon to launch it.
* On narrow screens every window opens maximised.

### Global shortcuts

| Keys | Action |
| --- | --- |
| `Ctrl` `Space` | Open the application menu (with search across apps and files) |
| `Ctrl` `Alt` `T` | New terminal |
| `Ctrl` `W` | Close the focused window |
| `Alt` `Tab` | Cycle windows |
| `Esc` | Close menus |

## Terminal (ksh)

The shell reads and writes the same disk as every other app. Type `help` for the full list and `man <command>` for details.

**Syntax it understands:** pipes `|`, redirects `>` `>>` `<` `2>` `2>&1`, `&&`, `||`, `;`, variables `$VAR` and `${VAR:default}` style defaults, command substitution `$(cmd)`, arithmetic `$((1 + 2))`, globs `*` and `?`, single and double quotes, `for`, `while`, `until` and `if` blocks, history expansion `!!` and `!N`.

**Command groups:**

* **files:** `ls cd pwd cat tree mkdir rmdir touch rm mv cp find du df stat basename dirname realpath`
* **text:** `echo printf head tail wc grep sort uniq cut tr sed rev nl tee seq base64 sha256sum json xargs calc uuid cowsay`
* **code:** `js` (run a file, evaluate code, or open a REPL), `edit`, `sh`, `source`
* **net:** `curl wget weather open browse search`
* **system:** `ps kill run apps theme wall date cal uptime whoami hostname uname neofetch say lock pbcopy pbpaste`
* **shell:** `help man clear exit history alias unalias export unset env read test sleep time which true false`

**Keys:** `Tab` completes, `Up` and `Down` walk history, `Ctrl` `C` stops a running command, `Ctrl` `L` clears, `Ctrl` `R` searches history, `Ctrl` `D` closes.

`~/.kshrc` runs whenever a terminal opens, so put aliases and exports there.

Some things to try:

```
cd ~/Projects/scripts
js fib.js 40
sh hello.sh
tree ~
weather Hanoi
cowsay hello
```

## Code

* File tree on the left, tabs across the top, output panel at the bottom, preview on the right.
* Syntax highlighting for JavaScript, JSON, HTML (including inline script and style), CSS, Python, shell and Markdown.
* **Run** (`Ctrl` `Enter`) does the sensible thing for the file type:
  * JavaScript runs in a Web Worker with `console`, `process`, `require("fs")` and `require("path")` wired to the Kestrel disk. A runaway loop can be stopped with the Stop button.
  * HTML, CSS and Markdown open a live preview that updates as you type. Stylesheets, scripts and images referenced by relative path are pulled from the virtual disk, and links between your own pages work.
  * JSON is validated; shell scripts run in a Terminal.
* Errors in the output panel link back to the exact file and line.

| Keys | Action |
| --- | --- |
| `Ctrl` `S` | Save |
| `Ctrl` `Enter` | Run |
| `Ctrl` `P` | Go to file |
| `Ctrl` `F` | Find and replace (supports case matching and regex) |
| `Ctrl` `G` | Go to line |
| `Ctrl` `/` | Toggle comment |
| `Alt` `Up` / `Down` | Move lines (add `Shift` to duplicate) |
| `Ctrl` `[` / `]` | Outdent / indent |
| `Ctrl` `B` | Toggle the file tree |
| `Ctrl` `J` | Toggle the output panel |

Press **Open the sample site** on the empty editor screen to load the small demo website in `~/Projects` with preview on.

## Browser

* Type an address, a local path (`~/Projects/...` or `file:///...`), or search words.
* `kestrel://` pages: `home`, `manual`, `status`, `logbook`, `history`.
* Bookmarks and history are saved between visits.
* YouTube watch links are rewritten to the embeddable player, and Google gets the parameter it needs to allow framing.

**A note on real websites:** many large sites (GitHub, Reddit, DuckDuckGo, MDN and others) tell browsers not to display them inside another page. Kestrel recognises the common ones and offers an **Open in new tab** button instead of a blank frame. Wikipedia, Google, the Internet Archive, Open Library and NPR's text site work inside the window.

`Ctrl` `L` focuses the address bar, `Ctrl` `T` opens a tab, `Alt` `Left` / `Right` go back and forward, middle click closes a tab.

## Filesystem and saving

* The disk is a flat in memory map of paths to files and folders, starting at `/home/operator`.
* Every change is written back to browser storage (`localStorage` under the key `kestrel:v1`, or `window.storage` when the host page provides one), so your files, settings, shell history, bookmarks and high score survive reloads.
* Images from Paint are stored as PNG data URLs.
* If storage is unavailable (private browsing, blocked site data) everything still works for the session.
* **Settings, System, Erase and restart** wipes the disk and restores the factory files.

Factory layout:

```
/home/operator
  .kshrc
  Documents/   readme.txt, notes.txt
  Pictures/
  Projects/    README.md, todo.txt, a sample website, scripts/
/system/logs   boot.log
```

## How the code is organised

Everything is in one file: CSS design tokens and styles at the top, then a single script split into numbered sections.

| Section | Contents |
| --- | --- |
| 0 | DOM helpers and the SVG icon set |
| 1 | Config and persistence |
| 2 | Virtual filesystem (`resolve`, `mkdir`, `writeFile`, `rm`, `moveTo`, sample files) |
| 3 | Toasts and the modal prompt |
| 4 | Window manager: open, focus, minimise, maximise, snap, drag, resize, taskbar, launcher |
| 4b | Shared pieces: script runner worker, page builder for previews, Markdown renderer, syntax highlighter, the editor component |
| 5 | Terminal and the `ksh` lexer, parser and interpreter |
| 6 to 14 | The apps |
| 15 to 17 | Theme, desktop icons, start menu, context menu, clock, lock screen, global keys, boot |

### Adding an app

Register an object on `APPS` and add its id to `DESK_ICONS` if you want a desktop icon:

```js
APPS.hello = {
  name: "Hello", icon: "info", desc: "Says hello", w: 400, h: 240,
  mount(body, win, arg) {
    body.innerHTML = `<div class="pad"><h2 class="appt">Hello</h2></div>`;
    return () => { /* optional cleanup when the window closes */ };
  }
};
```

Useful options: `single: true` keeps one window per app, `reopen(win, arg)` handles relaunching it, `hidden: true` keeps it out of menus. Set `win.beforeClose` to an async function returning `false` to block closing (for unsaved changes).

### Adding a shell command

Inside the `KSH` module, call `def(names, group, usage, description, run)`. The `run` function gets a context with `args`, `out`, `err`, `read`, `input`, `path`, `stdin` and more, and returns an exit code.

## Limitations

* There is no Python runtime; `.py` files highlight but don't run.
* `curl`, `wget` and `weather` only reach sites that allow cross origin requests.
* Clipboard commands depend on the browser granting permission.
* Storage is per browser and per origin, and browsers typically cap it around 5 MB.

## Accessibility and preferences

* Respects `prefers-reduced-motion`, and animation can be turned off in Settings.
* Light and dark chrome, five wallpapers and six accent colours.
* Visible focus rings on all controls.
