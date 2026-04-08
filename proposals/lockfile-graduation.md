# [Proposal] Graduate Dev Container Lockfile from Preview to Stable

## Summary

> **Note:** This proposal does **not** modify the Development Container Specification itself. The [lockfile specification](https://github.com/devcontainers/spec/blob/main/docs/specs/devcontainer-lockfile.md) already defines the `devcontainer-lock.json` file format and is already part of the published spec. This proposal requests a change to the **Dev Container CLI** (in the [devcontainers/cli](https://github.com/devcontainers/cli) repository) to graduate the lockfile feature from preview to stable. The proposal is submitted here because the spec repository is the central location for evaluating changes to the dev container ecosystem.

### What is a dev container lockfile?

A dev container lockfile (`devcontainer-lock.json`) records the exact resolved version, OCI digest, and SHA-256 integrity checksum for every [Feature](https://containers.dev/implementors/features/) referenced in a `devcontainer.json` file. Much like `package-lock.json` in npm or `Cargo.lock` in Rust, the lockfile provides:

- **Reproducibility** — Builds on different machines or at different times resolve to the exact same Feature artifacts.
- **Integrity verification** — If a Feature artifact changes unexpectedly (e.g., due to compromise or registry misconfiguration), the build fails instead of silently using the altered artifact.
- **Reviewable diffs** — When Features are upgraded, the lockfile shows exactly what changed.

### Current status: preview behind experimental flags

The lockfile feature was introduced as a preview feature in the Dev Container CLI ([PR #495](https://github.com/devcontainers/cli/pull/495)) behind `--experimental-lockfile` and `--experimental-frozen-lockfile` flags. It has been stable and in production use — notably by the [devcontainers/images](https://github.com/devcontainers/images) repository — but the experimental flags create adoption friction and signal that the feature is not fully supported.

### Proposed change

This proposal requests graduating the lockfile feature to stable by making lockfile generation the default behavior for `devcontainer build` and `devcontainer up`, similar to how `npm install` automatically generates a `package-lock.json`. Today, lockfile creation requires passing `--experimental-lockfile` or using a `touch` workaround. After this change, `devcontainer-lock.json` is created automatically on the first build and kept up to date as `devcontainer.json` changes.

This is a behavior change that could be considered a minor breaking change: builds that previously produced no lockfile will now produce an additional `devcontainer-lock.json` file. In practice, this is low-impact — no existing builds will fail, no container output changes, and the only visible effect is a new file in the workspace. Users who don't want the lockfile can opt out with `--no-lockfile`. The experimental flags are preserved as hidden, deprecated aliases so existing CI pipelines continue to work without modification.

## Goal

Graduate the dev container lockfile from a preview feature to a first-class, stable feature of the Dev Container CLI with these objectives:

1. **Default to security** — Generate and update lockfiles automatically, matching the behavior of every major package manager.
2. **Preserve backward compatibility** — Provide a clean migration path from the experimental flags; justify any breaking change with a security rationale.
3. **Provide clear opt-out** — Users who don't want lockfile generation can explicitly opt out.

## Affected Commands

This proposal affects the following Dev Container CLI commands:

| Command | Change |
| ------- | ------ |
| `devcontainer build` | Add `--no-lockfile` and `--frozen-lockfile` flags. Enable lockfile by default. Deprecate experimental flags. |
| `devcontainer up` | Add `--no-lockfile` and `--frozen-lockfile` flags. Enable lockfile by default. Deprecate experimental flags. |
| `devcontainer outdated` | No change (already uses lockfile without flags). |
| `devcontainer upgrade` | No change (already writes lockfile without flags). |

## Background: Current CLI Behavior

The `build` and `up` commands currently support `--experimental-lockfile` and `--experimental-frozen-lockfile` flags (both `boolean`, default `false`, hidden). The `outdated` and `upgrade` commands already work with lockfiles without any flags.

### When is the lockfile read?

If a lockfile exists, it is used to pin and verify Features during `build` and `up` — no flag is currently required for this. Features are fetched by digest (not tag), and builds fail if the digest doesn't match. The experimental flags only control whether the lockfile is *written*, not whether it is *read*.

### When is the lockfile written?

| Condition | Result |
| --------- | ------ |
| No lockfile exists, no flags | Not created (this is the gap we're closing) |
| No lockfile exists, `--experimental-lockfile` | Created |
| Lockfile exists, `devcontainer.json` unchanged | No write (content identical) |
| Lockfile exists, `devcontainer.json` changed | Updated automatically |
| `--experimental-frozen-lockfile` and lockfile missing/mismatched | Error |

**Important:** Upstream feature releases do NOT trigger lockfile updates. The lockfile pins by digest, so new versions published under the same tag are invisible until `devcontainer upgrade` is run.

## Proposal

### 1. Auto-generate lockfile by default (`build`, `up`)

**Change:** When `build` or `up` processes Features and no lockfile exists, create one automatically.

**Rationale:** This is the single most impactful change. It makes lockfiles the default for all users, aligning with the npm/yarn/cargo pattern where `install` always produces a lockfile. Since the lockfile is a new file written alongside `devcontainer.json`, this does not change the container build output or break any existing workflow — it only adds a file.

**Detail:** Remove the condition that skips lockfile creation when no lockfile exists and no experimental flag is set.

### 2. Add `--no-lockfile` and `--frozen-lockfile` flags (`build`, `up`)

**Change:** Add `--no-lockfile` and `--frozen-lockfile` boolean flags to `build` and `up`.

| Flag | Behavior |
| ------ | ---------- |
| `--no-lockfile` | Skip both reading and writing the lockfile. Features resolve from the registry by tag as if no lockfile exists. The lockfile on disk (if any) is neither consulted nor modified. Matches `npm install --no-package-lock` / `pnpm install --no-lockfile` semantics. |
| `--frozen-lockfile` | Require lockfile to exist and match exactly; fail otherwise. |

**Detail:**

- `--no-lockfile` is the escape hatch for users who don't want lockfile behavior at all. It matches npm/pnpm convention.
- `--frozen-lockfile` replaces `--experimental-frozen-lockfile` with identical semantics.

### 3. Deprecate experimental flags (`build`, `up`)

**Change:** Keep `--experimental-lockfile` and `--experimental-frozen-lockfile` as hidden aliases. Emit a deprecation warning when they are used.

**Rationale:** This avoids breaking existing CI/CD pipelines immediately while guiding users to the new flags.

**Detail:**

- When `--experimental-lockfile` is passed, treat it as a no-op (lockfile is already the default) and log: `Warning: --experimental-lockfile is deprecated. Lockfiles are now enabled by default.`
- When `--experimental-frozen-lockfile` is passed, map it to `--frozen-lockfile` and log: `Warning: --experimental-frozen-lockfile is deprecated. Use --frozen-lockfile instead.`
- Plan to remove the experimental flags in a future major version. Announce the timeline in the changelog.

### 4. No changes to other commands

The `outdated`, `upgrade`, `exec`, `read-configuration`, and other commands require no changes.

## Breaking Changes

| Change | Impact | Justification |
| -------- | -------- | --------------- |
| Lockfile auto-generated on `build`/`up` | A new `devcontainer-lock.json` file appears in the workspace. Users may see new untracked files in git. | Security by default. The lockfile provides integrity verification for all Feature artifacts. Users who don't want it can pass `--no-lockfile`. This mirrors the behavior of every major package manager. |
| `--experimental-lockfile` triggers deprecation warning | CI logs include a new warning line. | Standard deprecation practice. No functional change. |

These are low-risk changes. No existing container build will fail. No existing lockfile will be invalidated. However, any CI/CD processes that enforce a lack of changes during build (e.g., by checking for a clean git state) may need to be updated to allow the new lockfile or the deprecation warning.

## Migration Guide

| Current usage | Migration |
| --------------- | ----------- |
| `devcontainer build --experimental-lockfile` | `devcontainer build` (lockfile is now default) |
| `devcontainer up --experimental-lockfile` | `devcontainer up` (lockfile is now default) |
| `devcontainer build --experimental-frozen-lockfile` | `devcontainer build --frozen-lockfile` |
| `devcontainer up --experimental-frozen-lockfile` | `devcontainer up --frozen-lockfile` |
| `touch .devcontainer-lock.json` (Codespaces workaround) | No change needed; still works. But the lockfile is now created automatically, so the touch workaround is no longer necessary. |
| No lockfile usage | Lockfile is created automatically. Commit it to source control for reproducibility. Pass `--no-lockfile` to opt out. |

## Documentation

The following user-facing documentation is in scope for this change:

- **CLI help text** — The `--no-lockfile` and `--frozen-lockfile` flags on `build` and `up` need visible, non-hidden descriptions in the yargs command definitions. The deprecated `--experimental-lockfile` and `--experimental-frozen-lockfile` flags remain hidden.
- **README.md** — The command list in the README does not currently include `outdated` or `upgrade`. These commands are already shipped and should be added to the command list. The README should also briefly mention that lockfiles are generated by default on `build` and `up`.
- **CHANGELOG.md** — Add an entry documenting the lockfile graduation, new flags, and deprecation of experimental flags. Follow the existing format (month/year header, version number, bulleted descriptions with PR links).

## Out of Scope

The following items are explicitly out of scope for this proposal:

- **Changes to the lockfile specification** — The [lockfile specification](https://github.com/devcontainers/spec/blob/main/docs/specs/devcontainer-lockfile.md) is already published and does not need to change for this proposal.
- **New `devcontainer.json` properties** — Adding `lockfile` or `locked` properties to `devcontainer.json` (as suggested in [Discussion #237](https://github.com/orgs/devcontainers/discussions/237#discussioncomment-16265065)) would require a spec change. This is a good idea for a future iteration, but out of scope here. The goal is to remove the experimental flags from the CLI without a spec change.
- **VS Code extension changes** — The VS Code `dev.containers.experimentalLockfile` setting is owned by the VS Code Dev Containers extension. Coordination is needed, but not a blocker — the CLI changes are backward compatible, and the extension can continue passing `--experimental-lockfile` until it is updated separately.

## Related

- [Lockfile specification](https://github.com/devcontainers/spec/blob/main/docs/specs/devcontainer-lockfile.md)
- [CLI implementation tracking issue](https://github.com/devcontainers/cli/issues/564)
- [CLI lockfile implementation PR #495](https://github.com/devcontainers/cli/pull/495)
- [Community discussion #237](https://github.com/orgs/devcontainers/discussions/237)
