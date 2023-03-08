# Composite Features

Related: https://github.com/devcontainers/spec/issues/109

## Motivation

We've seen significant interest in the ability to "reuse" or "extend" a given Feature with one or more additional Features.  Often in software a given tool depends on another (or several) tool(s)/framework(s).  As the dev container Feature ecosystem has grown, there has been a growing need to reduce redundant code in Features by utilizing first installing a set of dependant Features.

## Goal

The solution shall provide a way to publish a Feature that "depends" on >= 1 other pubished Features. Dependent Features will be installed by the ochestrating tool, with the version and order set by the author if necessary.

The solution outlined shall not only execute the installation scripts, but also merge the additional development container config, as outlined in the documented [merging logic.](https://containers.dev/implementors/spec/#merge-logic)

A non-goal is to require the use or implementation of a full-blown dependency management system (such as `npm` or `apt`).  The solution should not encourage authorship of individual Features that do not continue to operate as "self-contained, shareable units of installation code and development container configuration"[(1)][https://containers.dev/implementors/features/].  

Composing Features should eliminate provide an alternative to existing community solutions, code duplication, and "hacky" means of installing a dependent Feature before another.

## Existing community solutions

### @danielBraun89 + '@devcontainers-contrib'

This community repository containing 100+ Features provides a custom solution for dependencies, introducing an additional `feature-definition.json` file - superset of the `devcontainer-feature.json` with a [`dependencies` object](https://github.com/devcontainers-contrib/features/blob/db45f607e733f3d560f6527d89b6a9a85b3b806c/feature_definitions/elixir-asdf/feature-definition.json#L29-L50).  Their [custom CLI has a tool named `featmake`](https://github.com/devcontainers-contrib/cli/blob/main/resources/featmake/featmake.sh#L145-L149) that will use the `oras` tool to pull and execute the `install.sh` of the given Feature.  This strategy doesn't appear to merge in the other dev container configuration properties that a Feature may declare.

### Direct curl

We've seen instances where users directly `curl` a Feature's `install.sh` script to `bash`.  This strategy also doesn't merge in the other dev container configuration properties that a Feature may declare.

### Additional inspiration

Inspiration was taken from [this spec issue on the topic](https://github.com/devcontainers/spec/issues/109), the repositories listed above, [the buildpack specification](https://docs.cloudfoundry.org/buildpacks/understand-buildpacks.html), and [VS Code extension packs](https://code.visualstudio.com/blogs/2017/03/07/extension-pack-roundup).

## Specification

Introduce a new file type `devcontainer-feature.composite.json` with the following properties.

| Property | Type | Description |
|----------|------|-------------|
| `id` | `string` | The ID of the Feature.  This follows the same semantics of the `id` property in the `devcontainer-feature.json` file. |
| `version` | `string` | The version of the Feature.  This follows the same semantics of the `version` property in the `devcontainer-feature.json` file. |
| `dependsOn` | `array` | An array of objects (in installation order) that define the Features that this Feature depends on. |
| `dependsOn.id` | `string` | The ID of the Feature that this Feature depends on. |
| `dependsOn.version` | `string` | The version of the Feature that this Feature depends on. |
| `dependsOn.detect` | `string` | A command that will be executed to determine if the Feature should be installed.  If the command returns a non-zero exit code, the Feature will be installed.  If the command returns a zero exit code, the remaining install steps will be skipped. |
| `dependsOn.options` | `object` | An object of key/value pairs that will be passed to the Feature's `install.sh` script. |

#### Example `devcontainer-feature.composite.json`

```jsonc
{
    "id": "ghcr.io/devcontainers/features/foo",
    "version": "1.0.0",
    "dependsOn": [
        {
            "id": "ghcr.io/devcontainers/features/a",
            "version": "sha256:45b23dee08af5e43a7fea6c4cf9c25ccf269ee113168c19722f87876677c5cb2", // SHA of the published artifact is OK
            "detect": "a --version && cat /etc/a/.markerfile", // Only install this Feature is detect returns non-zero
            "options": {
                "bar": true
            }
        },
        {
            "id": "ghcr.io/devcontainers/features/b",
            "version": "1.2.3",  // An exact version is required.  We do not permit pinning to a major or minor version.
            "detect": undefined, // Omit or set as 'undefined' to always install this Feature.
            "options": {
                "zip": "zap"
            }
        }
    ]
}

```


