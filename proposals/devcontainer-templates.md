# Dev Container Templates reference

Development container "Templates" are self-contained, shareable units of development container configuration. The name comes from the idea that adding one of them allows you to quickly get your project up and running with a containerized environment. The Templates describe the appropriate container image, [Features](https://github.com/devcontainers/spec/blob/main/proposals/devcontainer-features.md), runtime arguments for starting the container, and VS Code extensions that should be installed. Each provides a container configuration file (devcontainer.json) and other needed files that you can drop into any existing folder as a starting point for containerizing your project.

Template metadata is captured by a `devcontainer-template.json` file in the root folder of the Template.

## Folder structure 

A Template is a self contained entity in a folder with at least a `devcontainer-template.json` and [`devcontainer.json`](/docs/specs/devcontainer-reference.md#devcontainerjson).  Additional files are permitted and are packaged along side the required files.

```
+-- template
|    +-- devcontainer-template.json
|    +-- .devcontainer
|        +-- devcontainer.json 
|        +-- (other files)
|    +-- (other files)
```

## devcontainer-template.json properties

The `devcontainer-template.json` file defines information about the Template to be used by any supporting tools.

The properties of the file are as follows:

| Property | Type | Description |
| :--- | :--- | :--- |
| `id` | string | ID of the Template. The `id` should be unique in the context of the repository/published package where the Template exists and must match the name of the directory where the `devcontainer-template.json` resides. |
| `version` | string | The semantic version of the Template. |
| `name` | string | Name of the Template. |
| `description` | string | Description of the Template. |
| `documentationURL` | string | Url that points to the documentation of the Template. |
| `licenseURL` | string | Url that points to the license of the Template. |
| `image` | string | #TODO |
| `options` | object | #TODO |
| `keywords` | array | List of strings relevant to a user that would search for this Template. |
| `categories` | object | #TODO |
| `platforms` | object | |
| `type` | string | |
