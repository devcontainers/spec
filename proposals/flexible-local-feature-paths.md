# Flexible Local Feature Paths

ref: https://github.com/devcontainers/spec/issues/671

## Motivation

Users who want to manage a collection of their own common dev container configurations and Features in a single repository face significant friction with the current specification. The current model requires local Features to be stored within the `.devcontainer/` folder at the project workspace folder root.

**Problems with the current approach:**

1. **Cannot share Features across configurations** - Local Features must be duplicated in each `.devcontainer/` folder
2. **Incompatible with VS Code's alternative configuration folders** - When using alternative dev container configuration paths, local Features in those locations aren't found
3. **Workarounds are cumbersome** - Users resort to hardlinking files or Git submodules to work around restrictions

**Benefits of local Features** that users want to preserve:

- Avoid building/publishing full-blown Feature packages for personal use
- Changes take effect on next build without additional steps
- Keep Features private without usage quotas/hosting costs

## Goal

Provide a way for users to specify a custom base directory for resolving local Feature paths, enabling Feature reuse across dev container configurations.

## Proposal: Add `localFeaturesRoot` Property

Add a new `localFeaturesRoot` property to `devcontainer.json` that specifies the base directory for resolving local Feature paths.

### Property Definition

| Property            | Type   | Description                                                                                                                                                                                                                                                                                                                           |
| ------------------- | ------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `localFeaturesRoot` | string | Optional path (relative to the folder containing `devcontainer.json`) that serves as the base for resolving local Feature paths. When specified, all local Feature paths (those prefixed with `./`) are resolved relative to this directory instead of the folder containing `devcontainer.json`. Defaults to `.` (current behavior). |

### Constraints

- The path must be relative (no absolute paths allowed)
- The resolved path must exist and be accessible during the build
- The path may use `..` to traverse upward from the `devcontainer.json` location

### Examples

#### Example 1: Shared Features in Parent Directory

Repository structure:

```
my-workspace/
├── shared-features/
│   └── my-shell-config/
│       ├── devcontainer-feature.json
│       └── install.sh
└── projects/
    └── project-a/
        └── .devcontainer/
            └── devcontainer.json
```

`devcontainer.json`:

```jsonc
{
  "name": "Project A",
  "image": "mcr.microsoft.com/devcontainers/base:ubuntu",
  "localFeaturesRoot": "../../../shared-features",
  "features": {
    "./my-shell-config": {} // Resolves to shared-features/my-shell-config
  }
}
```

#### Example 2: Centralized Feature Repository

Repository structure:

```
dev-container-configs/
├── features/
│   ├── python-tools/
│   │   ├── devcontainer-feature.json
│   │   └── install.sh
│   └── node-tools/
│       ├── devcontainer-feature.json
│       └── install.sh
└── configs/
    ├── python-project/
    │   └── devcontainer.json
    └── node-project/
        └── devcontainer.json
```

`configs/python-project/devcontainer.json`:

```jsonc
{
  "name": "Python Development",
  "image": "mcr.microsoft.com/devcontainers/python:3",
  "localFeaturesRoot": "../../features",
  "features": {
    "./python-tools": {}
  }
}
```

`configs/node-project/devcontainer.json`:

```jsonc
{
  "name": "Node Development",
  "image": "mcr.microsoft.com/devcontainers/javascript-node:20",
  "localFeaturesRoot": "../../features",
  "features": {
    "./node-tools": {}
  }
}
```

### Backward Compatibility

This proposal is fully backward compatible:

- The `localFeaturesRoot` property is optional
- When not specified, behavior is identical to current spec (local Features resolve relative to the `devcontainer.json` folder)
- Existing configurations continue to work without modification

### Resolution Algorithm

When resolving a local Feature path (one starting with `./`):

1. If `localFeaturesRoot` is specified:
   - Resolve `localFeaturesRoot` relative to the folder containing `devcontainer.json`
   - Resolve the Feature path relative to the resolved `localFeaturesRoot`
2. If `localFeaturesRoot` is not specified:
   - Resolve the Feature path relative to the folder containing `devcontainer.json` (current behavior)

### Security Considerations

- **No absolute paths**: Requiring relative paths prevents access to arbitrary filesystem locations
- **Build-time resolution**: Paths are resolved at build time within the context of the dev container build
- **User-controlled scope**: Users have full control over what paths are accessible via this property
