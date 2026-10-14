# 1337-config

**Your whole dev setup on any 1337 workstation, in one command.**

```text
        workstation A                              workstation B (tomorrow)
  ┌──────────────────────────────┐          ┌──────────────────────────────┐
  │  $HOME      5 GB   follows   │          │  $HOME      the same one     │
  │  /goinfre  50 GB   wiped     │          │  /goinfre   empty            │
  └──────────────┬───────────────┘          └──────────────▲───────────────┘
                 │  1337-logout: push                      │  1337-login: clone
                 ▼                                         │
  ┌────────────────────────────────────────────────────────┴──────────────┐
  │                    GitHub  ·  your private repos                      │
  └───────────────────────────────────────────────────────────────────────┘
```

**Contents:**
[Install](#install) ·
[1337-setup](#1337-setup) ·
[GitHub sign-in](#github-sign-in) ·
[Every day](#every-day) ·
[1337-repo](#1337-repo) ·
[What goes where](#what-goes-where) ·
[Synced folders](#synced-folders) ·
[Templates](#templates) ·
[Keywords](#keywords) ·
[Safe by design](#safe-by-design) ·
[Troubleshooting](#troubleshooting)

---

## Install

Paste this in a terminal:

```sh
git clone https://github.com/mohammedhrima/1337-config.git /goinfre/$USER/1337-config && /goinfre/$USER/1337-config/1337 setup && exec zsh
```

```text
  git clone ──▶ /goinfre/$USER/1337-config        the runner (a single Python file)
      │
  1337 setup ──▶ asks you questions ──▶ ~/1337.yml        your settings
      │                             └─▶ ~/.zshrc          the 1337-* commands
      │
  exec zsh ──▶ new shell: 1337-login, 1337-logout, 1337-repo, 1337-setup ready
```

You do this once. `$HOME` follows you, so every workstation has your settings
and commands. After a logout wipes goinfre, `1337-login` clones the runner again.

What setup adds at the end of `~/.zshrc` (it asks first; running it again
replaces this block, never adds a second one):

```sh
# >>> 1337-config >>>
# written by `1337 setup`, which rewrites everything between these two lines
export PATH="/goinfre/$USER/1337-config:$PATH"
1337-get()    { [ -d "/goinfre/$USER/1337-config" ] || git clone -q https://github.com/mohammedhrima/1337-config.git "/goinfre/$USER/1337-config"; }
1337-setup()  { 1337-get && 1337 setup && eval "$(1337 env)"; }
1337-login()  { 1337-get && 1337 login "$@"; eval "$(1337 env)"; }
1337-logout() { 1337 logout; }
1337-repo()   { 1337-get && 1337 repo "$@" && eval "$(1337 env)"; }
[ -x "/goinfre/$USER/1337-config/1337" ] && eval "$(1337 env)"
# <<< 1337-config <<<
```

```text
  1337-get       clone the runner into goinfre, if a logout wiped it
  1337-login     1337-get, then 1337 login, then load the new PATH into this terminal
  last line      every new terminal finds your tools
```

Needs `git`, `curl`, `/usr/bin/python3` with PyYAML: all on 1337 machines.

---

## 1337-setup

```text
┌   1337   setup
│
◇  Sign in to GitHub?                         ──▶ see GitHub sign-in below
│  yes
◇  GitHub username                            ──▶ your repos: github.com/<username>/…
│  mohammedhrima
◇  GitHub email, for your commits             ──▶ git config user.email, at every login
│  you@mail.com
◇  Relocate: $HOME folders moved to goinfre   ──▶ section relocated
│  cache, npm, vscode, code-data …
◇  Sync: $HOME folders kept in a private repo ──▶ section synced
│  google-cache, vscode-config
◇  Tools: offered at every login              ──▶ section tools
│  code, node, uv
◇  Extras                                     ──▶ optional login / logout steps
│  docker
◇  Add a repo? enter when done                ──▶ section repos (same as 1337-repo add)
│  ura-lang
│
◆  files
│  ✔ ~/1337.yml   written; the old one is ~/1337.yml.bak
│  ✔ ~/.zshrc     has the 1337 commands
◆  github
│  ✔ ssh            GitHub knows your key
│  ✔ google-cache   created on GitHub, private
│  ✔ vscode-config  created on GitHub, private
└  done. Next: 1337-login
```

Checkbox questions:

```text
◆  Tools: offered at every login  3 of 9
│  ❯ ◼ code       VS Code
│    ◼ node       Node.js and npm
│    ◼ uv         Python, its versions and packages
│    ◻ android    Android SDK
│    ◻ flutter    Flutter and Dart        + ninja, android
└  ↑/↓ move   space toggle   a all   enter confirm
```

| Key | Does |
|---|---|
| `↑` `↓` (or `k` `j`) | move |
| `space` | tick / untick |
| `a` | tick all / none |
| `enter` | confirm |
| `Ctrl-C` | cancel, nothing written |

**Run it again anytime.** Your current answers are already ticked, your repos
are kept as they are, and the old file is saved as `~/1337.yml.bak`.

---

## GitHub sign-in

Setup signs in to GitHub so it can create your private repos and add your SSH
key. No password, no extra tool: you click a link.

```text
   terminal                         browser                          GitHub
  ──────────                       ─────────                        ────────
  1337-setup
  open  https://github.com/login/device ──▶ page opens
  code  ABCD-1234                          you type ABCD-1234 ──────▶ approve?
  waiting…                                                            │ yes
     ◀──────────────────────────────────────────────── token ─────────┘
  ✔ signed in as mohammedhrima
     │
     └─▶ saved in ~/.config/1337/github-token   (only you can read it)
```

| The token is used to | When |
|---|---|
| create google-cache and vscode-config, private | `1337-setup` |
| create a missing synced repo, private | `1337-login`, without asking |
| create a missing repo of yours, private | `1337-repo add` and `1337-login`, after asking |
| add your SSH key to GitHub | `1337-setup` |
| read your email, as the default answer | `1337-setup` |

Say **no** to sign-in and setup still works: it prints a link for each repo to
create by hand (`github.com/new?name=…&visibility=private`) and shows your SSH
key to paste at `github.com/settings/ssh/new`.

**SSH key:** repos are cloned and pushed over SSH.

```text
  ~/.ssh/id_ed25519.pub, id_ecdsa.pub or id_rsa.pub exists?
     ├─ yes ─▶ use it
     └─ no  ─▶ make ~/.ssh/id_ed25519
                  │
  GitHub knows it? (ssh -T git@github.com)
     ├─ yes ─▶ ✔ done
     └─ no  ─▶ signed in? ─ yes ─▶ added to GitHub for you
                         └ no  ─▶ shown, with the link to paste it
```

---

## Every day

| Command | Does |
|---|---|
| `1337-login` | set up this session; asks what you need (`--all`: no questions) |
| `1337-logout` | push everything, clean `/goinfre` |
| `1337-repo add <repo>` | add a repo, clone it now ([below](#1337-repo)) |
| `1337 install flutter` | install a tool now, with what it needs |
| `1337-setup` | answer the questions again |
| `1337` | list the commands |

```text
  1337-login                                     1337-logout
  ──────────                                     ───────────
  identity   git name and email                  chrome      quit Chrome
  relocated  ~/.cache … ─▶ goinfre    ☑ pick     vscode      quit VS Code
  tools      install code, node …     ☑ pick     extensions  save the list of VS Code extensions
  chrome     quit Chrome, first time only        push        commit + push synced and repos
  vscode     quit VS Code, first time only       unlink      remove the symlinks from $HOME
  synced     clone + link Chrome, VS Code        docker      stop docker, delete its images
  repos      clone your code          ☑ pick     wipe        empty /goinfre/$USER, except docker/
  extensions install VS Code extensions ☑ pick
```

`☑ pick`: a checkbox question, everything ticked, `enter` to go on. Each step
is in `~/1337.yml`, under `login:` and `logout:`, in this order.

A step that fails stops the ones after it, unless it has `ignore_errors: true`.

### What a login looks like

```text
┌   1337   login
│
◇  relocated
│  cache, npm, vscode, code-data
│
│  ✔ cache  ~/.cache → /goinfre/you/relocated/.cache
│  · npm    ~/.npm already linked
│  ✔ relocated
│
◇  tools
│  code, node, uv
│
│  ● code  installing                    ● = working
│  ✔ code  installed                     ✔ = done
│  · node  already installed             · = nothing to do
│  ✔ tools
│
◆  synced
│  ✔ google-cache   cloned
│  ✔ google-cache   ~/.config/google-chrome → /goinfre/you/synced/google-cache
│  ✔ synced
│
◇  repos
│  ura-lang, paperdesk
│
│  ✔ ura-lang   up to date with origin
│  ▲ paperdesk  branch dev not updated   ▲ = warning, goes on
│  ✔ repos
│
└  login done
```

### What a logout looks like

```text
┌   1337   logout
│
◆  push
│  ✔ google-cache   pushed as a single commit
│  ✔ vscode-config  pushed
│  ✔ ura-lang       pushed
│  · paperdesk      nothing new to push
│  ▲ ~/…/workspace/test  is a git repo that ~/1337.yml does not list: it is not pushed
│  ✔ push
│
◆  unlink
│  ✔ cache  ~/.cache unlinked
│  ✔ unlink
│
◆  wipe
│  ✔ wipe
│
└  logout done
```

If a push fails, logout stops before `wipe`: nothing is lost.

```text
◆  push
│  ✖ ura-lang  your commits conflict with origin's; the rebase was undone, merge them by hand
│  ✖ push failed
│
└  logout stopped at push                ✖ = failed: fix it, run 1337-logout again
```

### 1337 install

Install a tool in the middle of a session. What it `needs` comes first.

```text
  1337 install flutter

  flutter ──needs──▶ ninja      installed first
          └─needs──▶ android    installed first
  then flutter

  1337 install            no name: the tools checkbox
  1337 install --all      every tool, no question
```

New tools are on `PATH` in new terminals. In this one: `eval "$(1337 env)"`.

### 1337 env

Prints the variables and `PATH` from `~/1337.yml`. Your `~/.zshrc` runs it at
every new terminal, so tools are found.

```text
$ 1337 env
export GITHUB_USER=mohammedhrima
export GOINFRE=/goinfre/mhrima
export TOOLS=/goinfre/mhrima/tools
export ANDROID_HOME=/goinfre/mhrima/tools/android
export PATH=/goinfre/mhrima/tools/code/bin:/goinfre/mhrima/tools/node/bin:…
```

---

## 1337-repo

Add a repo without editing `~/1337.yml`. **No link needed**: its name is enough.

| You type | Means | Written as `source:` |
|---|---|---|
| `1337-repo add ura-lang` | your repo | `git@github.com:<you>/ura-lang.git` |
| `1337-repo add OussamaDX/pharmaflow` | someone else's | `git@github.com:OussamaDX/pharmaflow.git` |
| `1337-repo add https://github.com/OussamaDX/pharmaflow` | link from the browser | `git@github.com:OussamaDX/pharmaflow.git` |
| `1337-repo add git@github.com:you/x.git` | SSH URL | the same |
| `1337-repo add https://gitlab.com/you/x.git` | another host | kept as it is |
| `1337-repo add a b c` | several | one entry each |

`<you>` is your GitHub username from setup. GitHub links become SSH URLs, so
the push at logout needs no password.

```text
  1337-repo add ura-lang
     │
     ├─▶ ~/1337.yml   repos:
     │                    ura-lang:
     │                        source: git@github.com:<you>/ura-lang.git
     │                        dest: $WORKSPACE/ura-lang
     │
     ├─▶ on GitHub?   ── no, and it is yours ──▶ "Create it, private?" (when signed in)
     │
     └─▶ cloned now   /goinfre/$USER/workspace/ura-lang, every branch
```

```text
$ 1337-repo
┌   1337   repos
│
│  · ura-lang   git@github.com:mohammedhrima/ura-lang.git   cloned
│  · paperdesk  git@github.com:mohammedhrima/paperdesk.git
└  add one: 1337-repo add <name | owner/name | URL>
```

| Command | Does |
|---|---|
| `1337-repo` | list your repos, with `cloned` next to the ones on this machine |
| `1337-repo add <repo>…` | add and clone |
| `1337-repo remove <name>…` | take out of `~/1337.yml`; the clone stays, but is not pushed at logout |

A name already used, or a change that would break `~/1337.yml`: refused,
nothing changed. Your comments in the file are kept.

---

## What goes where

```text
  $HOME  (only symlinks)          /goinfre/$USER                GitHub (private)

  ~/.cache, ~/.npm …      ───▶    relocated/                    —              emptied at logout
                                  tools/code, node …            —              installed again at login
  ~/.config/google-chrome ───▶    synced/google-cache    ◀──▶   google-cache
  ~/.config/Code/User     ───▶    synced/vscode-config   ◀──▶   vscode-config
                                  workspace/ura-lang     ◀──▶   ura-lang
```

| Section of `~/1337.yml` | Holds | After logout |
|---|---|---|
| `variables` | names used everywhere as `$NAME` | — |
| `relocated` | big `$HOME` folders: caches | gone |
| `tools` | programs | gone, installed again at login |
| `synced` | `$HOME` folders you want back: Chrome profile, VS Code settings | on GitHub |
| `repos` | your code, every branch and tag | on GitHub |
| `login`, `logout` | the steps, in order | — |

Files 1337 writes outside goinfre:

| File | What |
|---|---|
| `~/1337.yml` | your settings |
| `~/1337.yml.bak` | the previous one, after `1337-setup` |
| `~/.zshrc` | the 1337 lines, between `# >>> 1337-config >>>` and `# <<< 1337-config <<<` |
| `~/.config/1337/github-token` | the GitHub sign-in |
| `~/.ssh/id_ed25519` | only when you had no SSH key |

---

## Synced folders

A synced folder lives in a private repo. `$HOME` only holds a symlink to it.

```text
  first 1337-login       the repo is empty
                         ~/.config/google-chrome (your current profile)
                              └─ moves into ─▶ synced/google-cache
                         .gitignore written: only the small files that matter
                         ~/.config/google-chrome ─▶ symlink to it

  1337-logout            committed and pushed to GitHub, on main

  any workstation        1337-login clones it, links it: you are signed in again
```

| Synced item | Holds | Repo | Its `.gitignore` keeps out |
|---|---|---|---|
| `google-cache` | Chrome bookmarks, history, passwords, cookies | one commit, force-pushed (`squash: true`) | everything but 15 small files (not 1.8 GB of cache) |
| `vscode-config` | VS Code `settings.json`, keybindings, snippets, `extensions.txt` | normal history | `workspaceStorage/`, `globalStorage/`, `History/`, `sync/` |

Every state the repo can be in, at login:

```text
  repo on GitHub?
   ├─ no ──▶ signed in? ─ yes ─▶ ✔ created, private, empty ─┐
   │                    └ no  ─▶ ✖ link to create it shown   │   other repos go on
   │                                                         │
   ├─ yes, empty ◀───────────────────────────────────────────┘
   │     ├─ $HOME folder there? ─▶ moved into the repo
   │     ├─ .gitignore from `gitignore:` written
   │     └─ setup (if any) waits for a first commit
   │
   └─ yes, with files
         ├─ $HOME folder there? ─▶ moved next to it as <name>.replaced, until logout
         └─ cloned, linked
```

At logout:

| The repo… | `1337` |
|---|---|
| is still empty, nothing in it | `· nothing new to push`, logout goes on |
| got its first files | first commit on `main`, pushed |
| has changes | commit, push |
| has nothing new | `· nothing new to push` (no push at all) |

Chrome and VS Code are closed before their folder is linked or pushed, so
their files are not half written. Setup warns if one of these repos is public.

---

## Templates

Copy into `~/1337.yml`, under the section in the title.

<details>
<summary><b>repos:</b> a repo (or just run <code>1337-repo add name</code>)</summary>

```yaml
    my-project:
        source: git@github.com:you/my-project.git
        dest: $WORKSPACE/my-project
```

Built after cloning, with the tools it needs:

```yaml
    my-extension:
        source: git@github.com:you/my-extension.git
        dest: $WORKSPACE/my-extension
        needs:
            - node
        setup:
            - npm install
```

</details>

<details>
<summary><b>tools:</b> a tool from a <code>.tar.gz</code></summary>

```yaml
    go:
        source: https://go.dev/dl/go1.24.0.linux-amd64.tar.gz
        dest: $TOOLS/go
        path:
            - $TOOLS/go/bin
        env:
            GOPATH: $RELOCATED/go
        setup:
            - curl -fL -o go.tar.gz "$SOURCE"
            - tar -xzf go.tar.gz
            - mv go "$DEST"
```

`setup` runs in an empty temporary folder and must create `$DEST`.

</details>

<details>
<summary><b>relocated:</b> move a <code>$HOME</code> folder to goinfre</summary>

```yaml
    gradle:
        link: $HOME/.gradle
        dest: $RELOCATED/.gradle
```

</details>

<details>
<summary><b>synced:</b> keep a <code>$HOME</code> folder in a private repo</summary>

```yaml
    firefox-cache:
        source: git@github.com:$GITHUB_USER/firefox-cache.git
        dest: $SYNCED/firefox-cache
        link: $HOME/.mozilla/firefox
        squash: true                      # big binary files: keep one commit
```

Signed in, the next `1337-login` creates the repo on GitHub, private. Add
`gitignore:` lines to keep big files out of it.

</details>

<details>
<summary><b>login / logout:</b> a step of your own</summary>

```yaml
    wallpaper:
        run:
            - gsettings set org.gnome.desktop.background picture-uri "file://$HOME/wall.png"
        ignore_errors: true
```

</details>

<details>
<summary><b>.gitignore</b> for a Chrome profile repo you made before (a new one gets it by itself)</summary>

```gitignore
/*
!/.gitignore
!/Local State
!/First Run
!/Default/
!/Profile */
/*/*
!/*/Bookmarks
!/*/Bookmarks.bak
!/*/Preferences
!/*/Secure Preferences
!/*/History
!/*/Favicons
!/*/Top Sites
!/*/Shortcuts
!/*/Web Data
!/*/Login Data
!/*/Cookies
!/*/Custom Dictionary.txt
```

</details>

Every item and step, with comments: [1337.example.yml](1337.example.yml).

---

## Keywords

```text
  variables     NAME: value, used everywhere as $NAME
  relocated ┐
  tools     ├── items:  source · dest · link · setup · path · env · needs
  synced    │           + squash · gitignore   (synced and repos)
  repos     ┘
  login     ┐
  logout    ┴── steps:  run · do · choose · options · message · ignore_errors
```

Variables build on each other. Any value can use them, plus `$HOME` and `$USER`:

```text
  GOINFRE:   /goinfre/$USER        ─▶  /goinfre/mhrima
  TOOLS:     $GOINFRE/tools        ─▶  /goinfre/mhrima/tools
  dest:      $TOOLS/node           ─▶  /goinfre/mhrima/tools/node
```

How one item works, the `node` tool:

```text
    node:
        source: https://nodejs.org/…/node-v$NODE_VERSION-linux-x64.tar.xz   ─▶ $SOURCE
        dest: $TOOLS/node                                                    ─▶ $DEST
        path:
            - $TOOLS/node/bin            ─▶ added to PATH
        setup:                           ─▶ runs once, when $DEST does not exist yet
            - curl -fL -o node.tar.xz "$SOURCE"
            - tar -xJf node.tar.xz
            - mv node-v$NODE_VERSION-linux-x64 "$DEST"
```

| Item keyword | Means | Example |
|---|---|---|
| `source` | where it comes from | download URL, git URL |
| `dest` | where it lives on goinfre | `$TOOLS/node` |
| `link` | `$HOME` path replaced by a symlink to `dest` | `$HOME/.npm` |
| `setup` | lines run once, after `dest` is made (`$SOURCE`, `$DEST` set) | `make install` |
| `path` | folders added to `PATH` | `$TOOLS/node/bin` |
| `env` | variables exported | `ANDROID_HOME: ...` |
| `needs` | items set up first | flutter needs `android` |
| `squash` | one commit, force-pushed | Chrome profile |
| `gitignore` | lines of the `.gitignore` a new, empty repo starts with | `- /workspaceStorage/` |

| Step keyword | Means |
|---|---|
| `run` | lines run as one bash script |
| `do` | built-in: `link` (relocated) · `install` (tools) · `sync` (synced) · `clone` (repos) · `push` · `unlink` |
| `choose` | `true`: ask with checkboxes |
| `options` | with `run` + `choose`: lines printing the choices → `$CHOSEN`, `$SKIPPED` |
| `message` | commit message of `do: push` |
| `ignore_errors` | `true`: a failure does not stop the rest |

---

## Safe by design

What logout does with each repo:

```text
  stash, detached HEAD, or a merge half done? ── yes ─▶ ⛔ stop, nothing wiped
     │ no
  git add -A, commit "autosync on logout"
     │
  fetch, rebase on origin ── conflict ─▶ rebase undone ─▶ ⛔ stop, nothing wiped
     │ ok
  fast-forward the other branches that are only behind origin
     │
  push every branch and tag ── fails ─▶ ⛔ stop, nothing wiped
     │ ok
     ✔ ─▶ next repo … then unlink, then wipe
```

| At logout, a repo with… | `1337` |
|---|---|
| uncommitted changes, new branches, tags | ✅ commits and pushes them |
| a branch someone else pushed to | ✅ fast-forwards it, then pushes |
| nothing new (a teammate's repo you can only read) | ✅ no push, logout goes on |
| an empty repo, still empty | ✅ no push, logout goes on |
| commits that conflict with origin | ⛔ stops, **nothing wiped** |
| changes in a repo you cannot push to | ⛔ stops, **nothing wiped**, says why |
| stashes, detached HEAD, half-done merge | ⛔ stops, **nothing wiped** |
| a git repo in `$WORKSPACE` not in `~/1337.yml` | ⚠️ warns: not pushed |

| Also | |
|---|---|
| a mistake in `~/1337.yml` | every problem listed, nothing runs |
| a failed download, or Ctrl-C | the half-installed tool is removed; the next login retries |
| a synced repo missing on GitHub | created, private (signed in), or a link to create it |
| a real folder where a synced folder goes | moved next to it as `<name>.replaced`, not deleted |
| a folder inside a relocated one (`~/.config/Code/User`) | moved along, not deleted |
| a synced folder that fails to clone | the other synced folders and your repos still go on |
| a logout cut in the middle of a squash push | the next logout finishes it on the right branch |
| `/goinfre/$USER/docker` | never deleted |

---

## Troubleshooting

A mistake in `~/1337.yml` stops everything before it starts, with where it is:

```text
■  /home/you/1337.yml has 2 problems
│  tools.node: missing 'dest'
│  repos.ura-lang.needs: 'nodejs' is not an item in relocated, tools, synced, repos
└  fix it and run 1337 again
```

| You see | Do |
|---|---|
| `command not found: 1337-login` | open a new terminal, or paste the install line again |
| `~/1337.yml does not exist yet` | `1337-setup` |
| `~/1337.yml has N problems` | fix the lines it names |
| a new tool is not found | `eval "$(1337 env)"`, or a new terminal |
| `GitHub still refuses your key` | add `~/.ssh/id_*.pub` at github.com/settings/ssh/new |
| `GitHub does not know your SSH key` | `1337-setup` |
| `not on GitHub; create it, private and empty: <link>` | open the link, click Create, `1337-login` again |
| `does not exist, or you cannot read it` | someone else's repo: check the name, ask for access |
| `push failed: ERROR: Permission to … denied` | a repo you cannot push to: move your changes elsewhere or drop them |
| `Chrome did not quit` | close Chrome: its profile is being pushed |
| `branch x not updated` | you have commits origin does not: merge by hand |
