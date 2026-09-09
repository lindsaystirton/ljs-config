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

**If you're on a Homebrew-built native Emacs (e.g. `emacs-plus`) with
native-compilation enabled**, make sure `gcc` and `libgccjit` are the
*same* version (`brew list --versions gcc libgccjit`) -- they're
built together and a mismatch produces bizarre, hard-to-place native
compiler errors (`ld: library '...' not found`) on every startup,
not just when installing something new. `brew upgrade gcc` if
they've drifted apart. `early-init.el` below also has a Homebrew/Xcode
linker-path fix that's needed alongside this -- see the audit, §44 and
§50, for the full story if you hit this (§50 covers a third missing
library, `emutls_w`, that can turn up even when `gcc`/`libgccjit`
already match).

**`pkg-config` and `enchant`, for spell-checking.** This config uses
[jinx](https://github.com/minad/jinx) rather than the older
flyspell/aspell combination, which builds a small native module
against `libenchant` the first time it loads. Install both via
Homebrew (`brew install pkg-config enchant`) before first launch --
`exec-path-from-shell` (loaded first, specifically so tools like this
are visible to GUI Emacs at all) takes care of the rest automatically
once they're installed.

**`cmake` and `libtool`, for the terminal (`vterm`).** `vterm` also
builds a small native module locally the first time it loads (this
one against `libvterm`, for real terminal emulation -- see
`ljs-config-shell.org`). Install both via Homebrew (`brew install
cmake libtool`) before first launch, same as the jinx prerequisites
above.

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

**Zotero, with the [Better BibTeX](https://retorque.re/zotero-better-bibtex/)
plugin, for bibliography management.** This config doesn't manage
your reference database itself -- see `ljs-config-bibliography.org`.
Zotero is the shared library (so co-authors who don't use Emacs or
LaTeX can still add references), Better BibTeX exports it to a `.bib`
file on disk, and Emacs just reads that file via `citar`.

**Note your username.** Open Terminal and run `whoami`. You'll want
this for the per-user customisation step below -- see [What's
Inside](#whats-inside).

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
`~/.emacs.d` (see the next section) -- these aren't tracked as files
in this repo, but their exact contents are given in full below, so
creating them is a copy-paste, not a favour to ask anyone.

## Installation

**1. Get Emacs.** Emacs 30 or later is required; this config is
currently developed and tested against Emacs 31. [Homebrew](https://brew.sh)
(`brew install emacs-plus` or similar) or [emacsformacosx.com](https://emacsformacosx.com/)
both work.

**2. Create `init.el` and `early-init.el`.** These two files belong
at the top level of `~/.emacs.d` (as siblings of the `ljs-config/`
directory you just cloned), and are what actually boots everything
else. Both are deliberately minimal -- everything that isn't required
to run at this exact point in Emacs's startup lives in `ljs-config.org`
and the modules it loads instead -- so rather than asking anyone to
track down a copy, here's the entire contents of each. Create the two
files and paste these in verbatim:

`~/.emacs.d/early-init.el`:

```elisp
;;; early-init.el --- Runs before init.el, before package.el, before the GUI frame  -*- lexical-binding: t; -*-
;;
;; Part of ljs-config, Lindsay Stirton's personal emacs configuration.
;;
;; Startup order is: this file, then package activation (skipped
;; below), then the frame, then init.el -- so anything here has to
;; make sense running before package.el or the frame exist. Font
;; selection used to live here too, to avoid a startup flicker, but
;; moved to ljs-config-appearance.org once the early font-availability
;; check proved unreliable this soon on macOS.

;; Native-comp toolchain fix, needed on a Homebrew-built native
;; Emacs (e.g. emacs-plus): gcc's native-comp driver can't link
;; without help finding three things -- its own top-level runtime
;; libraries, its version/target-triple-specific internal runtime
;; libraries (where things like `libemutls_w.a' actually live), and
;; macOS's own `-lSystem' (which lives in the Xcode SDK, not in
;; Homebrew's gcc at all). Without this you'll see native-comp
;; errors like "ld: library 'System' not found" or "ld: library
;; 'emutls_w' not found" on every startup. All three computed rather
;; than hardcoded, since none of them are stable across a machine,
;; Homebrew prefix, Xcode SDK version, or gcc version bump -- the
;; internal runtime directory is found with `find' rather than
;; assembled from a guessed version/target-triple path, so this
;; survives the next `gcc' bump the same way `lib/gcc/current'
;; (Homebrew's own stable symlink) already does.
(let* ((brew (or (getenv "HOMEBREW_PREFIX") "/opt/homebrew"))
       (sdk (string-trim (shell-command-to-string "xcrun --show-sdk-path")))
       (gcc-lib (concat brew "/opt/gcc/lib/gcc/current"))
       (gcc-internal (string-trim
                      (shell-command-to-string
                       (concat "dirname \"$(find " gcc-lib
                               " -name libemutls_w.a 2>/dev/null | head -1)\"")))))
  (setenv "LIBRARY_PATH"
          (concat gcc-lib ":" gcc-internal ":" sdk "/usr/lib")))

;; straight.el is the only package manager this config uses, so
;; package.el's own startup activation is disabled.
(setq package-enable-at-startup nil)

;; Raised early so GC doesn't slow down ljs-config.org's own module
;; loading further down.
(setq gc-cons-threshold 20000000)

;; Extra pixels below each line -- integer = pixels, float = scale
;; factor, nil = none. See `C-h v line-spacing'.
(setq-default line-spacing 0.06) ;; tuned for Pragmata Pro

;;; early-init.el ends here
```

`~/.emacs.d/init.el`:

```elisp
;;; init.el --- Where all the magic begins  -*- lexical-binding: t; -*-
;;
;; Part of ljs-config, Lindsay Stirton's personal emacs configuration.
;;
;; Kept minimal: this file's only job is getting from "Emacs just
;; started" to "Org can read ljs-config.org". Everything else lives
;; in ljs-config.org and the modules it loads, or, if it must run
;; before the frame exists, in early-init.el.

(setq dotfiles-dir user-emacs-directory)
(add-to-list 'load-path (expand-file-name
                         "lisp" (expand-file-name
                                 "org" (expand-file-name
                                        "src" dotfiles-dir))))

;; Started before ljs-config.org's module chain loads, so
;; `emacsclient' can still reach in even if something later fails.
(require 'server)
(unless (server-running-p)
  (server-start))

(setq org-replace-disputed-keys t)
(require 'org)
(org-babel-load-file (expand-file-name "./ljs-config/ljs-config.org" dotfiles-dir))

;;; init.el ends here
```

That's genuinely everything in both files -- nothing has been
trimmed from this listing for space. If either file's real content
ever changes, this section is the thing to update to match.

**3. Launch Emacs.** On first launch, `straight.el` will clone and
build every package this config declares -- this needs an internet
connection and will take a few minutes the first time. Every package
after that just loads from your local `~/.emacs.d/straight/` cache.
If a package fails to build on the first attempt, quit and relaunch
Emacs and it will usually pick up where it left off.

**4. (Optional) Add your own customisations.** Create a file named
after your own `user-login-name` (the `whoami` step above) -- e.g.
`ljs-config/yourname.org` -- and it'll be loaded automatically on
every startup. This is where your own bibliography paths,
`org-directory`, and any other personal, machine-specific settings
go, rather than into the shared config files. `ljs46.org` and
`ljs.org` in this repo are Lindsay's own, real examples of what one
looks like.

## What's Inside

`ljs-config.org` is the entry point: it bootstraps `straight.el`, then
loads each of the following in turn. Each one is a literate `.org`
file -- the prose above each code block explains the *why*, not just
the *what*, so if a setting looks unusual, check there before
assuming it's a bug.

| File | What it configures |
|---|---|
| `ljs-config-packages.org` | Package declarations -- what to install, fetched and pinned by straight.el |
| `ljs-config-spelling.org` | Spell-checking via [jinx](https://github.com/minad/jinx) |
| `ljs-config-defuns.org` | Small utility functions used elsewhere in the config |
| `ljs-config-appearance.org` | Theme, modeline, fonts, frame behaviour |
| `ljs-config-completion.org` | Vertico + Consult + Orderless + Marginalia + Embark (minibuffer completion) and Corfu + Cape (in-buffer completion) |
| `ljs-config-discoverability.org` | which-key (keybinding hints), casual (Transient menus for Calc/Info/isearch), and avy (jump-to-visible-text navigation) |
| `ljs-config-bibliography.org` | Zotero + Better BibTeX as the shared reference library, `citar` for citation completion/insertion in AUCTeX (works alongside RefTeX, which still handles labels and cross-references) |
| `ljs-config-latex.org` | AUCTeX, RefTeX, Biber, and the SyncTeX/PDF-pane workflow |
| `ljs-config-stats.org` | R, ESS, and Stan (`stan-mode`, `company-stan`, `flycheck-stan`) |
| `ljs-config-text.org` | Markdown, Pandoc, CSV, and general text-file handling |
| `ljs-config-dired.org` | Dired extras, iBuffer's saved filter groups, and casual-dired's Transient menu |
| `ljs-config-git.org` | Magit and Forge |
| `ljs-config-org.org` | Org-mode, Org-roam, and the shared PDF/frame-splitting logic used by both Org and LaTeX |
| `ljs-config-shell.org` | Shell/terminal: eshell for everyday use, vterm (+ vterm-toggle) for anything needing a real terminal |
| `ljs-config-python.org` | Python via Elpy/ESS and Jupyter |
| `ljs-config-lisp.org` | Emacs Lisp editing conveniences |

**Personal, per-user overrides** live outside this list, in a file
named after your `user-login-name` -- `ljs46.org` and `ljs.org` in
this repo, one per machine Lindsay actually uses (see the audit,
§41). `ljs-config.org` loads whichever one matches automatically, and
silently loads nothing if none matches -- so on a fresh clone, or
someone else's machine, create `<your-username>.org` next to these
and put your own destination paths there (a bibliography location,
an `org-directory`) rather than editing any of the files above.

One further `.org` file exists in the repo but isn't currently loaded
by anything -- `ljs-config-bindings.org` (custom keybindings,
including `expand-region`, `multiple-cursors`, and the
silver-searcher search integration). Reviving it properly, rather
than re-enabling it as-is, is on the to-do list -- see the audit
notes.

## Known limitations (if you're not Lindsay)

This config is further along than a typical personal dotfiles repo --
every package is declared once, installs cleanly via `straight.el`,
the whole thing restarts without errors, and the per-user override
mechanism actually works -- but it hasn't yet had every last thing
done to it that would make it a true drop-in kit for someone else,
the way ESKSS was. Concretely, as of this writing:

- **A couple of custom keybindings described in the code aren't
  actually active** -- `ljs-config-bindings.org` looks fully
  configured but is never loaded, so don't be surprised if a binding
  mentioned in a comment somewhere doesn't do anything yet.
- **Elpy's Python interpreter has no per-user override yet**
  (`ljs-config-python.org`) -- it falls back to whatever
  `python3`/`jupyter` resolve to on `PATH`. That's a deliberate
  choice for now (nothing forces a specific virtualenv), not a gap
  like the bibliography path used to be -- but if you need a specific
  interpreter, that's exactly the kind of thing your own
  `<your-username>.org` is for.

None of this affects day-to-day use on Lindsay's own machine, but if
you're setting this up fresh, expect a bit of manual path-fixing
until these are addressed.

## License

GPLv3 -- see `LICENSE`, inherited from ESKSS.
