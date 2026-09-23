# Oh My Pi Profile Collection

## Quick start

Clone the repository into Oh My Pi's standard profile directory:

```sh
mkdir -p ~/.omp
git clone https://github.com/noizbuster/omp-profiles.git ~/.omp/profiles
cd ~/.omp/profiles
```

After cloning, one command provisions every profile in `omp-profiles.txt` using the default paths (`~/.omp/agent` and `~/.omp/profiles`). Stop OMP first:

```sh
~/.omp/profiles/omp-profile-share setup
```

This creates any missing profile directories and shares their runtime state from `~/.omp/agent`. On success, the script prints a profile summary and copyable aliases for the selected shell config.

Add permanent shortcuts for every profile. This selects `~/.zshrc` for Zsh and `~/.bashrc` for Bash; the final line makes them available immediately and preserves them for new terminals.

```sh
rc_file="${ZDOTDIR:-$HOME}/.zshrc"
[ "${SHELL##*/}" = "bash" ] && rc_file="$HOME/.bashrc"

cat >> "$rc_file" <<'EOF'
# Oh My Pi profile shortcuts
alias ompf='omp --profile fast'
alias ompg='omp --profile glm'
alias ompb='omp --profile budget'
alias ompr='omp --profile grok'
alias omph='omp --profile hybrid'
alias ompo='omp --profile openai'
alias ompp='omp --profile poor'
EOF

. "$rc_file"
```

Start a profile with its shortcut, for example `ompf` for the fast profile or `omph` for the hybrid profile. See the table below for every shortcut.

Each entry in `omp-profiles.txt` names an `omp --profile` profile. `omp-profile-share sync` creates any missing profile directory and its ordinary, writable `agent/config.yml`; all other discovered runtime state is shared from `~/.omp/agent`.

Managed links use paths relative to `<profile>/agent` (for example, `../../../agent/agent.db`), so they remain valid on another machine when the repository is checked out at `~/.omp/profiles`.

## Profiles

| Profile | Shortcut |
| --- | --- |
| `fast` | `ompf` |
| `glm` | `ompg` |
| `budget` | `ompb` |
| `grok` | `ompr` |
| `hybrid` | `omph` |
| `openai` | `ompo` |
| `poor` | `ompp` |

## Run a profile without aliases

```sh
omp --profile fast
omp --profile glm
omp --profile budget
omp --profile grok
omp --profile hybrid
omp --profile openai
omp --profile poor
```

## Shared state and private configuration

`omp-profile-share` dynamically links non-private state from both `~/.omp/agent` and the non-structural top level of `~/.omp`.

Current `agent/` links include:

- `agent.db`, `history.db`, and `models.db`
- `agents`, `sessions`, and `terminal-sessions`
- `last-changelog-version`

Current profile-root links include:

- `auth-broker.token`, `gpu_cache.json`, and `install-id`
- `logs`, `plugins`, and `run`

Profile-root links use `../../<item>` and agent links use `../../../agent/<item>`, keeping a clone portable at `~/.omp/profiles`. The profile's `agent/` directory, `profiles/`, and `profile-share-backups/` remain structural local paths. `config.yml` is deliberately excluded: it is an ordinary file in each profile directory. SQLite `-wal`, `-shm`, and `-journal` sidecar files are never linked. If a default-agent item such as `blobs`, `memories`, `skills`, or `extensions` exists, the utility will link it on the next sync.

## Provision and validate

`omp-profiles.txt` is the canonical list. Without profile arguments, `setup`, `sync`, and `status` process every nonblank, noncomment entry; `detach` requires explicit profile names.

Stop all OMP processes before changing profile links. Then run:

```sh
omp-profile-share sync
omp-profile-share status
```

The utility protects existing state: profile-local directories merge into the default store without overwriting shared files, then their original form is moved under `~/.omp/profile-share-backups/<profile>/<timestamp>/`. `setup` and `sync` refuse to run while OMP is active unless `OMP_PROFILE_SHARE_ALLOW_RUNNING=1` is explicitly set; `detach` always requires OMP to be stopped.

For a one-profile rollback, first stop OMP, then run:

```sh
omp-profile-share detach glm
omp-profile-share status glm
```

`detach` replaces managed links with independent copies while preserving that profile's private `config.yml`.
