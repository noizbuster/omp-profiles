# Oh My Pi Profile Collection

## Quick start

Clone the repository into Oh My Pi's standard profile directory:

```sh
mkdir -p ~/.omp
git clone https://github.com/noizbuster/omp-profiles.git ~/.omp/profiles
cd ~/.omp/profiles
```

Add permanent shortcuts for every profile. This selects `~/.zshrc` for Zsh and `~/.bashrc` for Bash; the final line makes them available immediately and preserves them for new terminals.

```sh
rc_file="${ZDOTDIR:-$HOME}/.zshrc"
[ "${SHELL##*/}" = "bash" ] && rc_file="$HOME/.bashrc"

cat >> "$rc_file" <<'EOF'
# Oh My Pi profile shortcuts
alias ompf='omp --profile fast'
alias ompg='omp --profile glm'
alias ompb='omp --profile budget'
alias omps='omp --profile spark'
alias ompr='omp --profile grok'
alias omph='omp --profile hybrid'
EOF

. "$rc_file"
```

Start a profile with its shortcut, for example `ompf` for the fast profile or `omph` for the hybrid profile. See the table below for every shortcut.

Each directory in this repository is an `omp --profile` profile. Its `agent/config.yml` is a normal, writable, profile-specific file; all other discovered runtime state is shared from `~/.omp/agent`.

Managed links use paths relative to `<profile>/agent` (for example, `../../../agent/agent.db`), so they remain valid on another machine when the repository is checked out at `~/.omp/profiles`.

## Profiles

| Profile | Alias | Main model |
| --- | --- | --- |
| `fast` | `ompf` | GPT-5.6-Luna |
| `glm` | `ompg` | GLM-5.2 |
| `budget` | `ompb` | GLM-5.2 |
| `spark` | `omps` | GPT-5.3-Codex-Spark |
| `grok` | `ompr` | Grok-4.5 |
| `hybrid` | `omph` | Codex + GLM-5.2 + Grok |

`ompg` was requested for both `glm` and `grok`; a shell cannot define two aliases with the same name. It is assigned to `glm`, and the Grok alias is `ompr`.

## Run a profile without aliases

```sh
omp --profile fast
omp --profile glm
omp --profile budget
omp --profile spark
omp --profile grok
omp --profile hybrid
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

Stop all OMP processes before changing profile links. Then run:

```sh
omp-profile-share sync fast glm budget spark grok hybrid
omp-profile-share status fast glm budget spark grok hybrid
```

The utility protects existing state: profile-local directories merge into the default store without overwriting shared files, then their original form is moved under `~/.omp/profile-share-backups/<profile>/<timestamp>/`. `setup` and `sync` refuse to run while OMP is active unless `OMP_PROFILE_SHARE_ALLOW_RUNNING=1` is explicitly set; `detach` always requires OMP to be stopped.

For a one-profile rollback, first stop OMP, then run:

```sh
omp-profile-share detach glm
omp-profile-share status glm
```

`detach` replaces managed links with independent copies while preserving that profile's private `config.yml`.
