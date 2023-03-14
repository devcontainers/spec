# Composite Features

Related: https://github.com/devcontainers/spec/issues/109

## Motivation

We've seen significant interest in the ability to "reuse" or "extend" a given Feature with one or more additional Features.  Often in software a given tool depends on another (or several) tool(s)/framework(s).  As the dev container Feature ecosystem has grown, there has been a growing need to reduce redundant code in Features by utilizing first installing a set of dependant Features.

## Goal

The solution shall provide a way to publish a Feature that "depends" on >= 1 other pubished Features. Dependent Features will be installed by the ochestrating tool, with the version and order set by the author if necessary.

The solution outlined shall not only execute the installation scripts, but also merge the additional development container config, as outlined in the documented [merging logic.](https://containers.dev/implementors/spec/#merge-logic)

A non-goal is to require the use or implementation of a full-blown dependency management system (such as `npm` or `apt`).  The solution should not encourage authorship of individual Features that do not continue to operate as "self-contained, shareable units of installation code and development container configuration"[(1)](https://containers.dev/implementors/features/).  

Composing Features should provide an alternative to existing community solutions, code duplication, and "hacky" means of installing a dependent Feature before another.

## Definitions

- `Standalone Feature` - The existing dev container Feature format, as outlined in the [spec](https://containers.dev/implementors/features/), and defined by a `devcontainer-feature.json` metadata file. A Feature that is self-contained, shareable unit of installation code and development container configuration.
- `Composite Feature` - A Feature that depends on >= 1 other Features, defined by a `devcontainer-feature.composite.json` metadata file.

## Existing community solutions

### @danielBraun89 + '@devcontainers-contrib'

This community repository containing 100+ Features provides a custom solution for dependencies, introducing an additional `feature-definition.json` file - superset of the `devcontainer-feature.json` with a [`dependencies` object](https://github.com/devcontainers-contrib/features/blob/db45f607e733f3d560f6527d89b6a9a85b3b806c/feature_definitions/elixir-asdf/feature-definition.json#L29-L50).  Their [custom CLI has command named `install`](https://github.com/devcontainers-contrib/cli/blob/0768a6f9a75934e4915739ad3b43f6feb5ec515e/dcontainer/cli/install/install_feature.py) that will use python to pull and execute the `install.sh` of the given Feature.  This strategy doesn't merge in the other dev container configuration properties that a Feature may declare.

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
| `features` | `array` | An array of objects (in installation order) that define the Feature(s) that compose this Feature. |
| `features.id` | `string` | The ID of the Feature that this Feature depends on. |
| `features.version` | `string` | The version of the Feature that this Feature depends on. |
| `features.detect` | `string` | A command that will be executed in a shell to determine if the Feature should be installed.  If the command returns a non-zero exit code, the Feature will be installed.  If the command returns a zero exit code, the remaining install steps will be skipped. |
| `features.options` | `object` | An object of key/value pairs that will be passed to the Feature's `install.sh` script. |

#### Example `devcontainer-feature.composite.json`

```jsonc
{
    "id": "ghcr.io/devcontainers/features/composite",
    "version": "1.0.0",
    "features": [
        {
            "id": "ghcr.io/devcontainers/features/a", // Must be a standalone Feature. (A composite Feature cannot depend on a composite Feature).
            "version": "1.2.3",  // An exact version is required.  We do not permit pinning to a major or minor version.
            "detect": "a --version && cat /etc/a/.markerfile", // Only continue installation of this Feature if detect returns non-zero
            "options": {
                "bar": true
            }
        },
        {
            "id": "ghcr.io/microsoft/features/b",
            "version": "sha256:45b23dee08af5e43a7fea6c4cf9c25ccf269ee113168c19722f87876677c5cb2", // SHA of the published artifact is OK
            "detect": undefined, // Omit or set as 'undefined' to always install this Feature.
            "options": {
                "zip": "zap"
            }
        }
    ]
}
```

An optional `finalize.sh` script can be included, and will be fired after all Features have been installed.

A composite Feature will be published following the same process as an [standalone dev container Feature](https://containers.dev/implementors/features) into the same namespace - following the pattern outlined in [the Features distribution spec](https://containers.dev/implementors/features-distribution/). Dependencies of a composite Feature can be published to the same or different namespaces.

An example repository structure for a repo with one composite Feature and an standalone Feature can be found below:

```
$ tree 

├── src
│   ├── composite
│   │   ├── README.md
│   │   ├── devcontainer-feature.composite.json
│   │   ├── finalize.sh
│   ├── a
│   │   ├── README.md
│   │   ├── devcontainer-feature.json
│   │   └── install.sh
...
```

### Notes:

- Composite Features cannot depend on other composite Features. This is to prevent a circular dependencies and deep dependency chains.
- The `dependsOn` array is in the order that the Features should be installed.  This is important for Features that depend on the existence of a file or directory created by a previous Feature.
- The `detect` property is optional.  If omitted, the Feature will always be installed.
- The `options` property is optional.  If omitted, the default options will be passed to the Feature's `install.sh` script, as defined in the Feature's `devcontainer-feature.json` file.
- A composite Feature can optionally include a `finalize.sh` script.  This script will be executed after all of the dependencies have been installed.  This is useful for Features that need to perform some action after all of the dependencies have been installed.


## Advantages

- Composite Features can be used to distribute a single Feature that depends on other Features.
- Composite Features can pin all of their dependencies to a specific version, ensuring that the Feature can be tested and will work as expected.
- Composite Features prevent the complexity that arises with deeply nested dependencies or circular dependencies.

## Disadvantages

- Composite features are not as flexible as other possible dependency models.
