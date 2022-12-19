Goal: Adds support for renaming and deprecating a Feature.

Adding two new properties to the [devcontainer-feature.json](../docs/specs/devcontainer-features.md#devcontainer-featurejson-properties) 👇 

# 1. legacyIds

| Property | Type | Description |
| :--- | :--- | :--- |
| `legacyIds` | array | Defaults to `[]`. Array of old IDs used to publish this Feature. The property is useful for renaming a currently published Feature within a single namespace. |

- The [source code](../docs/specs/features-distribution/#source-code) can be changed to a new name along with [devcontainer-feature.json properties](../docs/specs/devcontainer-features.md#devcontainer-featurejson-properties) (like `id`, `name`, `documentationURL`).
- The [distribution tools](../docs/specs/features-distribution/#distribution) like [Dev Container CLI](https://github.com/devcontainers/cli) and [Dev Container Publish GitHub Action](https://github.com/marketplace/actions/dev-container-publish) will dual publish the old `id`s (which is defined by the `legacyIds` property) and the new `id`.
    - The [putManifestWithTags](https://github.com/devcontainers/cli/blob/main/src/spec-configuration/containerCollectionsOCIPush.ts#L172) will be modified. The same `tgz` file for the `id` will be pushed to the `id`s  mentioned by the `legacyIds` property for all the [tags](https://github.com/devcontainers/cli/blob/main/src/spec-configuration/containerCollectionsOCIPush.ts#L175).

Example: Let's say we currently have a `docker-from-docker` Feature 👇 

Current `devcontainer-feature.json` : 

```jsonc
{
    "id": "docker-from-docker",
    "version": "2.0.1",
    "name": "Docker (Docker-from-Docker)",
    "documentationURL": "https://github.com/devcontainers/features/tree/main/src/docker-from-docker",
    ....
}
```

We'd want to rename this Feature to `docker-outside-of-docker`. The source code folder of the Feature will be renamed to `docker-outside-of-docker` and the updated `devcontainer-feature.json` will look like 👇 

```jsonc
{
    "id": "docker-outside-of-docker",
    "version": "2.0.2",
    "name": "Docker (Docker-outside-of-Docker)",
    "documentationURL": "https://github.com/devcontainers/features/tree/main/src/docker-outside-of-docker",
    ....
}
```

**Note** - The semantic version of the Feature defined by the `version` property should be **continued** and should not be restarted at `1.0.0`. 

### Supporting backwards compatibility for [`installsAfter`](../docs/specs/devcontainer-features.md#2-the-installsafter-feature-property) property

- Currently the `installsAfter` property is defined as an array consisting of the Feature `id` that should be installed before the given Feature.
- The Feature to be renamed could be already defined by other Feature authors in their `installsAfter` property. Renaming the `id` could change the installation order for them if the `installsAfter` property is not updated with the new `id`. In order to avoid this unexpected behavior and to support back compat, the CLI tooling will be updated to also look at the `legacyIds` property along with the `id` for determining the installation order.
 
 ### Supporting tools
 
This property can be used by the supporting tools to indicate Feature rename in few ways 
 - [Dev Container Publish GitHub Action](https://github.com/devcontainers/action) which auto-generates the README files can add a note with a list of old `id`s which were used to publish this Feature.
 -  In the scenarios where Dev Configs are referencing the old `id`s,  the VS Code extension hints could be enhanced to deliver this warning that the `id` was renamed to a new one.
- The [containers.dev/features](https://containers.dev/features) website, and [supporting tools](https://containers.dev/supporting) like [VS Code Dev Containers](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers) and [GitHub Codespaces](https://github.com/features/codespaces) wouldn't list the old `id`, instead list the new `id` of Features.
 
# 2. deprecated

| Property | Type | Description |
| :--- | :--- | :--- |
| `deprecated` | boolean | Defaults to `false`. Indicates that the Feature is deprecated, and will not receive any further updates/support. |

- If this property is set to `true`, then it indicates that the Feature is deprecated and won't be receiving any further updates/support. The OCI artifacts would exist, hence, the current dev configs will continue to work.
- This property could be used by the supporting tools to indicate Feature deprecation in few ways -
    - [Dev Container Publish GitHub Action](https://github.com/devcontainers/action) which auto-generates the README files can be updated to add a top-level header mentioning it's deprecation
    - The [containers.dev/features](https://containers.dev/features) website, and [supporting tools](https://containers.dev/supporting) like [VS Code Dev Containers](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers) and [GitHub Codespaces](https://github.com/features/codespaces) wouldn't list the deprecated Features.
    -  VS Code extension hints could be enhanced to deliver this information about it's deprecation
   