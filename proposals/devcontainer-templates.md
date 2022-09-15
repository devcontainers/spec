# Dev Container Templates reference

Development container "Templates" are self-contained, shareable units of development container configuration. The name comes from the idea that adding a Template to your project helps it get up and running with a containerized environment. The Templates describe the appropriate container image, [Features](https://github.com/devcontainers/spec/blob/main/proposals/devcontainer-features.md), runtime arguments for starting the container, and VS Code extensions that should be installed. Each provides a container configuration file ([`.devcontainer.json`](/docs/specs/devcontainer-reference.md#devcontainerjson)) and other needed files that you can drop into any existing folder as a starting point for containerizing your project.

Template metadata is captured by a `devcontainer-template.json` file in the root folder of the Template.

## Folder structure 

A Template is a self contained entity in a folder with at least a `devcontainer-template.json` and [`devcontainer.json`](/docs/specs/devcontainer-reference.md#devcontainerjson).  Additional files are permitted and are packaged along side the required files.

```
+-- template
|    +-- devcontainer-template.json
|    +-- .devcontainer.json
|    +-- (other files)
```

## devcontainer-template.json properties

The `devcontainer-template.json` file defines information about the Template to be used by any [supporting tools](#template-supporting-tools-and-services).

The properties of the file are as follows:

| Property | Type | Description |
| :--- | :--- | :--- |
| `id` | string | ID of the Template. The `id` should be unique in the context of the repository/published package where the Template exists and must match the name of the directory where the `devcontainer-template.json` resides. |
| `name` | string | Name of the Template. |
| `description` | string | Description of the Template. |
| `documentationURL` | string | Url that points to the documentation of the Template. |
| `licenseURL` | string | Url that points to the license of the Template. |
| `type` | string | Type of the dev container (image, dockerfile, dockerCompose) created by the Template. |
| `options` | object | A map of options that the supporting tools should use to populate different configuration options for the Template. |
| `platforms` | object | Languages and platforms supported by the Template. |
| `keywords` | array | List of strings relevant to a user that would search for this Template. |

### The `options` property

The `options` property contains a map of option IDs and their related configuration settings. These `options` are used by the supporting tools to prompt the user to choose from different Template configuration options. The tools would replace the option ID with the selected value in the specified `replaceIn` files. This replacement would happen before dropping the `.devcontainer.json` (or `.devcontainer/devcontainer.json`) and other files (within the sub-directory of the Template) required to containerize your project. See [option resolution](#option-resolution) for more details. For example:

```json
{
  "options": {
    "optionId": {
      "type": "string",
      "description": "Description of the option",
      "proposals": ["value1", "value2"],
      "default": "value1",
      "replaceIn": [".devcontainer.json"],
    }
  }
}
```

| Property | Type | Description |
| :--- | :--- | :--- |
| `optionId` | string | ID of the option used by the supporting tools to replace the selected value in the specified `replaceIn` files. |
| `optionId.type` | string | Type of the option. Valid types are currently: `boolean`, `string` |
| `optionId.description` | string | Description for the option. |
| `optionId.proposals` | array | A list of suggested string values. Free-form values **are** allowed. Omit when using `optionId.enum`. |
| `optionId.enum` | array | A strict list of allowed string values. Free-form values are **not** allowed. Omit when using `optionId.proposals`. |
| `optionId.default` | string | Default value for the option. |
| `optionId.replaceIn` | array | List of file paths which the supporting tool should use to perform the string replacement of `optionId` with the selected value. The provided path is always relative to the sub-directory of the Template. |

> `Note`: The `options` must be unique for every `devcontainer-template.json`

### Template Supporting Tools and Services

Outlines tools and services that currently support adding Templates to a project.

- [Visual Studio Code Remote - Containers](../docs/specs/supporting-tools.md/#visual-studio-code-remote---containers)
- [GitHub Codespaces](../docs/specs/supporting-tools.md/#github-codespaces)

## Adding a Template to a project or codespace
  
  1. Either [create a codespace for your repository](https://aka.ms/ghcs-open-codespace) or [set up your local machine](https://aka.ms/vscode-remote/containers/getting-started) for use with the Remote - Containers extension, start VS Code, and open your project folder.
  2. Press <kbd>F1</kbd>, and select the **Add Development Container Configuration Files...** command for **Remote-Containers** or **Codespaces**.
  3. Pick one of the recommended Templates from the list or select **Show All Templates...** to see all of them. You may need to choose the **From a predefined container configuration Template...** option if your project has an existing Dockerfile or Docker Compose file. Answer any questions that appear.
  4. See the Template's `README` for configuration options.
  5. Run **Remote-Containers: Reopen in Container** to use it locally, or **Codespaces: Rebuild Container** from within a codespace.

### Referencing a Template 

The `id` format (`<oci-registry>/<namespace>/<template>[:latest]`) dictates how a [supporting tool](#template-supporting-tools-and-services) will locate and download a given Template from an OCI registry (For example - `ghcr.io/user/repo/go:latest`). The registry must implement the [OCI Artifact Distribution Specification](https://github.com/opencontainers/distribution-spec). Some implementors can be [found here](https://oras.land/implementors/). 

## Versioning

Each Template is published with only the `latest` tag. Tooling that handles releasing Templates will republish the `latest` tag every time a new release is created.

## Release

_For information on distribution Templates, see [devcontainer-templates-distribution.md](./devcontainer-templates-distribution.md)._

### Option Resolution

A templates's `options` is used by a supporting tool to prompt for different configuration options. A supporting tool will parse the `options` object provided by the user. If a value is selected for a Template, it will be replaced in the provided `replaceIn` files.

### Option resolution example

Consider a `java` template with the following folder structure:

+-- java
|    +-- devcontainer-template.json
|    +-- .devcontainer.json

Suppose the `java` Template has the following `options` parameters declared in the `devcontainer-template.json` file:

```json
// ...
"options": {
    "imageVariant": {
        "type": "string",
        "description": "Specify version of java.",
        "proposals": [
          "17-bullseye",
          "17-buster",
          "11-bullseye",
          "11-buster",
          "17",
          "11"
        ],
			  "default": "17-bullseye",
        "replaceIn": [".devcontainer.json"]
    },
    "nodeVersion": {
        "type": "string", 
        "proposals": [
          "latest",
          "16",
          "14",
          "10",
          "none"
        ],
        "default": "latest",
        "description": "Specify version of node, or 'none' to skip node installation.",
        "replaceIn": [".devcontainer.json"]
    },
    "installMaven": {
        "type": "boolean", 
        "description": "Install Maven, a management tool for Java.",
        "default": "false",
        "replaceIn": [".devcontainer.json"]
    },
}
```

and it has the following `.devcontainer.json` file:

```json
{
	"name": "Java",
	"image": "mcr.microsoft.com/devcontainers/java:0-${imageVariant}",
	"features": {
		"ghcr.io/devcontainers/features/node:1": {
			"version": "${nodeVersion}",
      "installMaven": "${installMaven}"
		}
	},
//	...
}
```

A user tries to add the `java` Template to their project using the [supporting tools](#template-supporting-tools-and-services) and selects `17-bullseye` when prompted for `"Specify version of Go."` and uses the `default` values for `"Specify version of node, or 'none' to skip node installation."` and `"Install Maven, a management tool for Java."`, then the modified `.devcontainer.json` (according to the `replaceIn` property) will be as follows:

```json
{
	"name": "Go",
	"image": "mcr.microsoft.com/devcontainers/go:0-17-bullseye",
	"features": {
		"ghcr.io/devcontainers/features/node:1": {
			"version": "latest",
      "installMaven": "false"
		}
	},
	...
}
```

The modified `.devcontainer.json` would be dropped into any existing folder as a starting point for containerizing your project.
