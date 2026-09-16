# Git Struggles

Practice Git recovery in real repositories. Ten replayable levels cover local
history mistakes, GitHub collaboration failures, and a missing object database
entry. Solve them with ordinary Git commands; the CLI checks the resulting
content, ancestry, working state, and (where required) actual remote history.

Requires **Python 3.10+**, **Git on PATH**, and a dedicated **GitHub repository**.
There are no runtime Python dependencies. Git for Windows is supported.

For a **no-install Windows launch**, run `git-struggles.cmd` in this folder.
In PowerShell, a session alias keeps the command available after changing into
an exercise directory:

```powershell
Set-Alias git-struggles (Join-Path (Get-Location) 'git-struggles.cmd')
git-struggles
```

Alternatively, run `python "<absolute-path-to-this-folder>\run.py" <command>`
from any directory. The launcher needs no pip, setuptools, or downloaded packages.

## Install and start

Open a terminal in this source folder:

```powershell
python -m pip install -e .
git-struggles
```

The first run asks for your GitHub repository and exercise folder. Create the
GitHub repository with a README first, so it has a permanent default branch.
Use a repository dedicated to these exercises. Leave the `struggles/**` branches
free of protection/rulesets that prohibit creation, force pushes, or deletion.

Authentication uses your existing SSH agent or Git credential helper. With
HTTPS and GitHub CLI installed, setup offers browser sign-in. You can also run
`gh auth login` followed by `gh auth setup-git` yourself. Git Credential Manager
is another supported route. Tokens/passwords are never written to app context.

Setup creates, rewrites, and deletes one uniquely named probe branch. It leaves
the default branch alone. If deletion is blocked, it reports the exact probe
branch needing cleanup. Fix permissions and rerun `git-struggles setup`.

```powershell
git-struggles levels
git-struggles start 01
# Run the cd command printed by the app.
git log --all --graph --oneline --decorate
# Investigate and repair with Git.
git-struggles check
```

If the installed command is not on PATH, use `python -m git_struggles` after
installation. For an isolated installation, create a virtual environment first:

```powershell
python -m venv .venv
.\.venv\Scripts\python.exe -m pip install -e .
.\.venv\Scripts\git-struggles.exe
```

The absolute executable path works after changing into an exercise folder.
Without installation, `python -m git_struggles` also works **from this source
folder**; it checks the active attempt when you are outside an exercise.

## Commands

| Command | Behavior |
|---|---|
| `setup` | Configure GitHub remote, authentication, and exercise root |
| `levels` / `list` | Show all levels and progress; nothing is locked |
| `start 04` | Resume the latest usable attempt of level 04, or create one |
| `replay 04` | Create a fresh attempt, retaining previous attempts |
| `status` | Repeat the current incident description and paths |
| `check` | Inspect completion; return 0 for success, 1 for incomplete, 2 for unavailable/error |
| `hint` | Reveal one more hint and show all hints revealed so far |
| `hint --show` | Show already revealed hints without advancing |
| `solution` | Explicitly reveal the complete repair approach |
| `restart` | Create a new attempt at the current level |
| `undo` | Restore the healthy pre-catastrophe baseline of the current attempt |
| `attempts` | List all attempts and their IDs |

`status`, `check`, `hint`, `solution`, `restart`, and `undo` accept
`--attempt <id>`. Otherwise the app selects the attempt containing your current
working directory; outside an exercise, it uses the saved active attempt.
**After restart, change directory to the newly printed path.** Staying in the
old directory intentionally continues to address the old attempt.

Progress is saved at `~/.git-struggles/context`, a versioned JSON file. Hints and
solution use are tracked per attempt; completion with help still counts. Replay
does not erase earlier completion. An optional global `--home <folder>` creates
a separate profile, for example `git-struggles --home D:\practice-state levels`.

## Levels

| ID | Exercise | Remote use |
|---|---|---|
| 01 | Three commits on the wrong branch | Local |
| 02 | Hard reset lost committed work | Local |
| 03 | Detached HEAD work abandoned | Local |
| 04 | Rejected push after a teammate's update | GitHub; publish repair |
| 05 | Bad rebase dropped work and resolved a conflict incorrectly | Local |
| 06 | Revert a published merge while retaining later work | GitHub; publish repair |
| 07 | Reintroduce a corrected, previously reverted feature | GitHub; publish repair |
| 08 | Move your commits off rewritten upstream history | GitHub; publish repair |
| 09 | Recover shared history erased by force push | GitHub and teammate clone; publish repair |
| 10 | Restore a missing blob without losing unpushed commits | GitHub and teammate clone; local repair |

Each level has three progressive hints and an explicit solution. The tiny Python
and text files are exercise material; you do not need additional libraries or
to run a service. Constraints are stated in each incident description.

## Remote branch mapping

Your local `main` tracks a remote branch such as
`struggles/<installation>/level-04/<attempt>/main`. The exercise's fetch refspec
maps it to the familiar `origin/main`. Use **plain `git push`** (or `git push
--force-with-lease` if a particular exercise permits rewriting).

**`git push origin main` explicitly targets the remote branch named `main`,
which is the repository's permanent branch, not your exercise branch.** If you
want an explicit refspec, use the full attempt prefix printed by `status`.
Use `git remote -v` and `git config --get-regexp 'branch\..*'` to inspect mapping.
The app itself only pushes to its exact attempt namespace.

All GitHub work uses your single account; different commit histories and clones
simulate teammates. No GitHub Actions workflows, pull requests, or account
creation are needed. Attempt directories and remote branches are retained for
inspection; this version does not automatically prune old attempts.

## Recovery and troubleshooting

- `restart` is the normal way to retry. It gives you a new directory and remote
  namespace, even if you have severely damaged the previous attempt.
- `undo` restores a snapshot from before `execute`, including that attempt's
  remote branch tips. It archives current local repositories under
  `before-undo-*` rather than deleting them. It asks for confirmation because it
  deliberately changes the existing attempt's remote history.
- Setup failure is recorded as `setup-failed`. Inspect the error and use
  `restart`; partial attempt files and remote branches are retained.
- A network/authentication error during remote checking gives UNKNOWN and never
  records completion. Restore connectivity and check again.
- A failed remote operation can have an uncertain outcome if the connection
  drops after the server processes it. Inspect the reported attempt namespace
  on GitHub; replay uses a new namespace and is safe to attempt independently.
- If a process was killed while holding `context.lock`, confirm it has exited
  before removing that lock. Ordinary failures release it automatically.
- Exercise `.gitattributes` normalizes Python, Markdown, and text files to LF
  when added to Git, so normal Windows line endings do not cause false failures.
  Binary object recovery still needs exact bytes.
- The app uses local identity/configuration and disables hooks and automatic
  garbage collection in the exercise repositories to keep recovery evidence.
  Your ordinary Git repositories and global configuration are not changed.

## Development and verification

```powershell
python -m unittest discover -s tests -v
```

Tests use real Git and temporary local bare remotes. They do not contact GitHub
or alter your normal app profile. Live GitHub authentication and repository
policy are checked by first-run setup, and require your actual repository.

Read [DEVELOPMENT.md](DEVELOPMENT.md) for architecture, the scenario contract,
step-by-step extension instructions, checker design, evidence, and limitations.
#   g i t _ s t r u g g l e s  
 