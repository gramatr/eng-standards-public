# Releasing

Versioning and release process for `eng-standards`.

## Versioning scheme

[Semantic Versioning](https://semver.org/) — `MAJOR.MINOR.PATCH`.

- **v0.x.y** while the standards are still evolving. Breaking changes allowed in minor bumps. Consuming repos should expect occasional friction.
- **v1.0.0** once standards stabilize enough that consuming repos shouldn't expect breaking changes within a major version.

We're at **v0.1.x** — too new to lock down.

## When to bump

| Bump | Trigger |
|---|---|
| **MAJOR** (`1.x.y → 2.0.0`) | Breaking change to a standards JSON shape that forces consuming repos to update their enforce-* scripts. |
| **MINOR** (`1.0.x → 1.1.0`) | New standard added (new domain). Non-breaking schema change. New rule added with a grace period. |
| **PATCH** (`1.0.0 → 1.0.1`) | Drift fixes, prose/template updates, generator regenerations, comment-only changes. |

During v0.x.y, both **MINOR** and **PATCH** can include breaking changes — that's the contract while pre-1.0. Be explicit in the tag annotation about what broke.

## Release process

1. Land all desired commits on `main` via PR (per `APPROVAL.md`).
2. Update `package.json` `version` to match the new tag (drift between tag and package.json is confusing; keep them in sync).
3. Open a tagging PR (or hand-merged commit on main) that bumps `package.json`.
4. Once merged, cut the tag at `main` HEAD:
   ```bash
   git tag -a vX.Y.Z -m "<release notes — what changed, who's affected, breaking changes>" main
   git push origin vX.Y.Z
   ```
5. Notify consuming repos. (Manual for now — future: bot opens PRs to bump submodule pins.)

Tag annotations are the canonical release notes. `git show vX.Y.Z` should give a reviewer the full picture of what's in the release.

## Consuming repos

Consuming repos **pin the submodule to a tag**, not to a branch or a commit hash:

```bash
# First-time add
git submodule add https://github.com/gramatr/eng-standards.git eng-standards
git -C eng-standards checkout v0.1.0
git add eng-standards
git commit -m "feat: pin eng-standards to v0.1.0"

# Update to a new tag
git -C eng-standards fetch --tags
git -C eng-standards checkout v0.1.1
git add eng-standards
git commit -m "chore: bump eng-standards to v0.1.1"
```

### Freshness rule (current)

`scripts/check-submodule.sh` currently checks commits-behind-main. **TODO** for a future PR: switch this to tags-behind-latest so the freshness rule matches the pinning rule.

## Tag-line convention

Annotation format:

```
v0.2.0 — adds <new standard name>

Highlights:
- New: standards/<name>.json (governs <domain>)
- Change: <breaking change description>, migration: <path>
- Fix: <drift or correction>

Affected consuming repos: <list from scope/*.yaml>
Migration window: <days, per the affected standard's migration phases>
```

Keep it short enough to read in `git tag -l --format='%(refname:short) — %(contents:subject)'`.

## What does NOT need a tag

- In-progress work on a feature branch
- Drift fixes that don't change observable behavior (e.g., regenerating JSON to match YAML when the YAML hasn't changed since the last tag) — these can wait until the next minor bump

## Tags are immutable

Once a tag is pushed, do not move or rewrite it. If a tagged release has a critical bug, cut a patch release (e.g., `v0.1.1`); never overwrite `v0.1.0`. Consuming repos that pinned to the broken tag stay stable.
