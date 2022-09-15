# Dev Container Templates reference

Development container "Templates" are self-contained, shareable units of development container configuration. The name comes from the idea that adding a Template to your project helps it get up and running with a containerized environment. The Templates describe the appropriate container image, [Features](https://github.com/devcontainers/spec/blob/main/proposals/devcontainer-features.md), runtime arguments for starting the container, and VS Code extensions that should be installed. Each provides a container configuration file ([`devcontainer.json`](/docs/specs/devcontainer-reference.md#devcontainerjson)) and other needed files that you can drop into any existing folder as a starting point for containerizing your project.

Template metadata is captured by a `devcontainer-template.json` file in the root folder of the Template.

## Folder structure 

A Template is a self contained entity in a folder with at least a `devcontainer-template.json` and [`devcontainer.json`](/docs/specs/devcontainer-reference.md#devcontainerjson).  Additional files are permitted and are packaged along side the required files.

```
+-- template
|    +-- devcontainer-template.json
|    +-- .devcontainer
|        +-- devcontainer.json 
|        +-- Dockerfile (optional)
|        +-- docker-compose.yml (optional)
|        +-- (other files)
|    +-- (other files)
```

## devcontainer-template.json properties

The `devcontainer-template.json` file defines information about the Template to be used by any [supporting tools](#template-supporting-tools-and-services).

The properties of the file are as follows:

| Property | Type | Description |
| :--- | :--- | :--- |
| `id` | string | ID of the Template. The `id` should be unique in the context of the repository/published package where the Template exists and must match the name of the directory where the `devcontainer-template.json` resides. |
| `version` | string | The semantic version of the Template. |
| `name` | string | Name of the Template. |
| `description` | string | Description of the Template. |
| `documentationURL` | string | Url that points to the documentation of the Template. |
| `licenseURL` | string | Url that points to the license of the Template. |
| `type` | string | Type of the dev container (image, dockerfile, dockerCompose) created by the Template. |
| `image` | string | Name of an image in a container registry ([DockerHub](https://hub.docker.com/), [GitHub Container Registry](https://docs.github.com/en/packages/learn-github-packages/introduction-to-github-packages), [Azure Container Registry](https://azure.microsoft.com/en-us/products/container-registry/)) that the [Template supporting services / tools](#template-supporting-tools-and-services) should use to find the image variants. |
| `features` | object | List of [Feature id](devcontainer-features.md/#referencing-a-feature) that the [Template supporting services / tools](#template-supporting-tools-and-services) should use to populate different options. |
| `options` | object | A map of options that will be passed as `build.args` when building a dockerfile. |
| `platforms` | object | Languages and platforms supported by the Template. |
| `keywords` | array | List of strings relevant to a user that would search for this Template. |

### The `options` property

The options property contains a map of option IDs and their related configuration settings. The ID becomes the name of the build arguments in all caps and snake case which is passed while building a dockerfile. For example:

```json
{
  "options": {
    "optionIdGoesHere": {
      "type": "string",
      "description": "Description of the option",
      "proposals": ["value1", "value2"],
      "default": "value1"
    }
  }
}
```

| Property | Type | Description |
| :--- | :--- | :--- |
| `optionId` | string | ID of the option that is converted into an all-caps and snake case build argument with the selected value in it. |
| `optionId.type` | string | Type of the option. Valid types are currently: `boolean`, `string` |
| `optionId.proposals` | array | A list of suggested string values. Free-form values **are** allowed. Omit when using `optionId.enum`. |
| `optionId.enum` | array | A strict list of allowed string values. Free-form values are **not** allowed. Omit when using `optionId.proposals`. |
| `optionId.default` | string or boolean | Default value for the option. |
| `optionId.description` | string | Description for the option. |

> **Note:** Options are helpful only when the dev container is built using a `dockerfile`.

### Template Supporting Tools and Services

Outlines tools and services that currently support adding Templates to a project.

- [Visual Studio Code Remote - Containers](../docs/specs/supporting-tools.md/#visual-studio-code-remote---containers)
- [GitHub Codespaces](../docs/specs/supporting-tools.md/#github-codespaces)

## Adding a Template to a project or codespace
  
  1. Either [create a codespace for your repository](https://aka.ms/ghcs-open-codespace) or [set up your local machine](https://aka.ms/vscode-remote/containers/getting-started) for use with the Remote - Containers extension, start VS Code, and open your project folder.
  2. Press <kbd>F1</kbd>, and select the **Add Development Container Configuration Files...** command for **Remote-Containers** or **Codespaces**.
  3. Pick one of the recommended Templates from the list or select **Show All Templates...** to see all of them. You may need to choose the **From a predefined container configuration Template...** option if your project has an existing Dockerfile or Docker Compose file. Answer any questions that appear.
  4. See the Template's `README` for configuration options. A link is available in the `.devcontainer/devcontainer.json` file added to your folder.
  5. Run **Remote-Containers: Reopen in Container** to use it locally, or **Codespaces: Rebuild Container** from within a codespace.

### Referencing a Template 

The `id` format (`<oci-registry>/<namespace>/<template>[:<semantic-version>]`) dictates how a [supporting tool](#template-supporting-tools-and-services) will locate and download a given Template from an OCI registry. The registry must implement the [OCI Artifact Distribution Specification](https://github.com/opencontainers/distribution-spec). Some implementors can be [found here](https://oras.land/implementors/). For example.

- `ghcr.io/user/repo/go`
- `ghcr.io/user/repo/go:1`
- `ghcr.io/user/repo/go:latest`

## Versioning

Each Template is individually [versioned according to the semver specification](https://semver.org/).  The `version` property in the respective `devcontainer-template.json` file is updated to increment the Template's version.

Tooling that handles releasing Templates will not republish Templates if that exact version has already been published; however, tooling must republish major and minor versions in accordance with the semver specification.

## Release

_For information on distribution Templates, see [devcontainer-templates-distribution.md](./devcontainer-templates-distribution.md)._
