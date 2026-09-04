# ljs-config

Lindsay Stirton's personal Emacs configuration: a "monster academic
machine" for research and writing in plain text, built around LaTeX,
R/ESS/Stan, Python, Org-mode, and reference management, all wired
together with [`use-package`](https://github.com/jwiegley/use-package)
and [`straight.el`](https://github.com/radian-software/straight.el).

Draws heavily -- to the point of shameless plagiarism -- on
[Kieran Healy's Emacs Starter Kit for the Social Sciences](https://kieranhealy.org/resources/emacs-starter-kit/)
(ESKSS), which is where this file's structure comes from too. If
you're a social scientist looking for a more actively maintained
starting point than the now-archived ESKSS, you could do worse than
fork this instead of starting from scratch.

> **Status.** This is a personal config, actively being rebuilt, not
> a polished drop-in kit yet. It runs cleanly end to end on Lindsay's
> own machine. If you're cloning this to use yourself (rather than
> just to read it), see [Known limitations](#known-limitations-if-youre-not-lindsay)
> below before you start -- a couple of things that ESKSS handled for
> every user aren't wired up here yet.

## Motivation

Emacs is extraordinarily powerful but not very useful out of the
box: most of the settings and modes that make it pleasant to write in
every day aren't switched on by default, and wiring up a proper
LaTeX+statistics+reference-management+version-control workflow by
hand is a lot of yak-shaving before you get to do any actual writing.
This repo is that yak-shaving, already done, so that starting Emacs
gets you straight to work rather than to a blank scratch buffer.

Compared with ESKSS, the aim here is to be a little more minimal and
opinionated, and to lean fully on `use-package` and `straight.el` --
both of which either didn't exist yet or weren't the obvious choice
when ESKSS was actively developed. Every package this config uses is
declared in exactly one place, is fetched and pinned by straight.el
rather than by the old, unpinnable `package.el`, and the reasoning
behind non-obvious settings is written up as prose in the `.org` file
next to the code, not left for you to reverse-engineer later.

## Platform

This config is macOS-only, and has only ever been run on Lindsay's own
Macs. It works across both Apple Silicon and Intel Macs (see the
architecture notes in the audit), but macOS itself is assumed
throughout, not just tested against: `exec-path-from-shell` (needed
because macOS GUI apps don't inherit your shell's `PATH`), a couple of
`(when (eq system-type 'darwin) ...)` checks, and Homebrew-based tool
lookups (for `stanc`, PDF tools, and others) are all woven through the
individual `.org` files rather than isolated in one place. None of it
has been tried on Linux or Windows, and getting it there would be real
work, not a config tweak.

## Before You Begin

If you want the tools this config wires together -- LaTeX, R, Stan,
Git, and the rest -- you'll need them installed on your Mac first:

**Xcode Command Line Tools.** Open Terminal and run:

```
xcode-select --install
```

**A modern TeX distribution and a PDF reader with SyncTeX support.**
[MacTeX](https://www.tug.org/mactex/) and the built-in
[pdf-tools](https://github.com/vedang/pdf-tools) (which this config
installs and configures for you) are what this config is built and
tested against.

**R and Stan**, if you're doing statistical work: [R](https://www.r-project.org/)
itself, plus `stanc`/`cmdstan` if you want Stan model checking and
compilation to work (`ljs-config-stats.org` expects `stanc` to be on
your `PATH`, or findable via Homebrew).

**Git.** You'll need it to clone this repo in the first place, and
the config assumes you're using [Magit](https://magit.vc/) day to day
rather than the command line.

**Note your username and hostname.** Open Terminal and run `whoami`
and `hostname`. You'll want these for the per-user customisation step
below -- see the caveat in [Known limitations](#known-limitations-if-youre-not-lindsay)
about the current state of that mechanism.

## Getting the Config

Clone it straight into `~/.emacs.d`. If you already have Emacs
configured, back up your existing `~/.emacs.d` first (`mv ~/.emacs.d
~/.emacs.d.bak`), then:

```
git clone git@github.com:lindsaystirton/ljs-config.git ~/.emacs.d/ljs-config
```

Note this clones the config into a `ljs-config/` subdirectory of
`~/.emacs.d`, not `~/.emacs.d` itself -- unlike ESKSS, this repo is
just the literate `.org` config files, not the whole `.emacs.d` tree.
You'll also need `init.el` and `early-init.el` at the top level of
`~/.emacs.d` (see the next section and the caveat below -- these
aren't in this git repo yet).

## Installation

**1. Get Emacs.** Emacs 30 or later is required; this config is
currently developed and tested against Emacs 31. [Homebrew](https://brew.sh)
(`brew install emacs-plus` or similar) or [emacsformacosx.com](https://emacsformacosx.com/)
both work.

**2. Put `init.el` and `early-init.el` in place.** These two files
belong at the top level of `~/.emacs.d` (as siblings of the
`ljs-config/` directory you just cloned), and are what actually boots
everything else:

- `early-init.el` disables `package.el`'s own startup activation, so
  it doesn't fight with `straight.el` (the only package manager this
  config uses).
- `init.el` sets up fonts and a couple of startup tweaks, then calls
  `(org-babel-load-file ".../ljs-config/ljs-config.org")`, which is
  what actually loads every module below.

For now these two files need to be copied over by hand (ask Lindsay,
or see [Known limitations](#known-limitations-if-youre-not-lindsay))
-- getting them into this repo, with a proper per-user override
mechanism, is the next piece of work planned for this project.

**3. Launch Emacs.** On first launch, `straight.el` will clone and
build every package this config declares -- this needs an internet
connection and will take a few minutes the first time. Every package
after that just loads from your local `~/.emacs.d/straight/` cache.
If a package fails to build on the first attempt, quit and relaunch
Emacs and it will usually pick up where it left off.

**4. (Optional) Add your own customisations.** Once the per-user
override mechanism described below is rebuilt, this is where you'll
be able to drop in your own bibliography paths, Python environment
paths, and other personal settings without editing the shared config
files directly.

## What's Inside

`ljs-config.org` is the entry point: it bootstraps `straight.el`, then
loads each of the following in turn. Each one is a literate `.org`
file -- the prose above each code block explains the *why*, not just
the *what*, so if a setting looks unusual, check there before
assuming it's a bug.

| File | What it configures |
|---|---|
| `ljs-config-elpa.org` | Package-list bootstrap (legacy name -- predates the straight.el migration) |
| `ljs-config-aspell.org` | Spell-checking (`flyspell`, `ispell`/`aspell`) |
| `ljs-config-defuns.org` | Small utility functions used elsewhere in the config |
| `ljs-config-appearance.org` | Theme, modeline, fonts, frame behaviour |
| `ljs-config-completion.org` | Vertico + Consult + Orderless + Marginalia + Embark (minibuffer completion) and Corfu + Cape (in-buffer completion) |
| `ljs-config-latex.org` | AUCTeX, RefTeX, ebib, Biber, and the SyncTeX/PDF-pane workflow |
| `ljs-config-stats.org` | R, ESS, and Stan (`stan-mode`, `company-stan`, `flycheck-stan`) |
| `ljs-config-text.org` | Markdown, CSV, and general text-file handling |
| `ljs-config-git.org` | Magit and Forge |
| `ljs-config-org.org` | Org-mode, Org-roam, and the shared PDF/frame-splitting logic used by both Org and LaTeX |
| `ljs-config-eshell.org` | Eshell configuration |
| `ljs-config-python.org` | Python via Elpy/ESS and Jupyter |
| `ljs-config-lisp.org` | Emacs Lisp editing conveniences |

Two further `.org` files exist in the repo but aren't currently
loaded by anything -- `ljs46.org` (a per-user override file) and
`ljs-config-bindings.org` (custom keybindings, including
`expand-region`, `multiple-cursors`, and the silver-searcher search
integration). Reviving these properly, rather than re-enabling them
as-is, is on the to-do list -- see the audit notes.

## Known limitations (if you're not Lindsay)

This config is further along than a typical personal dotfiles repo --
every package is declared once, installs cleanly via `straight.el`,
and the whole thing restarts without errors -- but it hasn't yet had
the specific things done to it that would make it a true drop-in kit
for someone else, the way ESKSS was. Concretely, as of this writing:

- **`init.el` and `early-init.el` aren't in this repo.** They live
  only on Lindsay's machine. If you clone `ljs-config/` alone, you
  won't have an entry point yet.
- **There's no working per-user override file.** ESKSS solved this by
  having you rename a template file to `%your-username%.org`; this
  config has the beginnings of the same idea (`ljs46.org`), but it
  isn't actually wired up to load, and personal absolute paths (a
  bibliography location, a Python virtualenv) are hardcoded into the
  shared files instead. If you fork this, expect to find and replace
  a small number of `/Users/ljs46/...`-style paths in
  `ljs-config-latex.org` and `ljs-config-python.org`.
- **A couple of custom keybindings described in the code aren't
  actually active** -- `ljs-config-bindings.org` looks fully
  configured but is never loaded, so don't be surprised if a binding
  mentioned in a comment somewhere doesn't do anything yet.
- **The startup frame size is hardcoded to Lindsay's own screen.**
  `ljs-config-appearance.org`'s `initial-frame-alist` pins every new
  frame to 85 columns by 54 rows at the screen's top-left corner --
  numbers tuned by eye to fill Lindsay's 13.3-inch MacBook display
  exactly, not computed from the actual screen. On a differently
  sized or positioned display this will either leave a lot of unused
  screen or not fit at all. `ljs/frame-double-width`, defined just
  below it in the same file, already does this properly -- it reads
  the real monitor size via `frame-monitor-workarea` rather than
  assuming one -- and doing the same for the initial frame size is on
  the to-do list rather than something to copy as-is.

None of this affects day-to-day use on Lindsay's own machine, but if
you're setting this up fresh, expect a bit of manual path-fixing
until these are addressed.

## License

GPLv3 -- see `LICENSE`, inherited from ESKSS.
