# Supporting tools and services

This page outlines tools and services that currently support the development container specification, including the `devcontainer.json` format. A `devcontainer.json` file in your project tells tools and services that support the dev container spec how to access (or create) a dev container with a well-defined tool and runtime stack. 

While most [dev container properties](devcontainerjson-reference.md) apply to any supporting tool or service, a few are specific to certain tools, which are outlined below.

## Editors

### Visual Studio Code

Visual studio code specific properties go under `vscode` inside `customizations`.


```jsonc
"customizations": {
		// Configure properties specific to VS Code.
		"vscode": {
			// Set *default* container specific settings.json values on container create.
			"settings": {},
			"extensions": [],
		}
}
```


| Property | Type | Description |
|----------|------|-------------|
| `extensions` | array | An array of extension IDs that specify the extensions that should be installed inside the container when it is created. Defaults to `[]`. |
| `settings` | object | Adds default `settings.json` values into a container/machine specific settings file. Defaults to `{}`. |

Please note that [Remote - Containers](#visual-studio-code-remote---containers) and [GitHub Codespaces](#github-codespaces) support the VS Code properties.

## Tools

### Dev container CLI

A dev container command line interface (CLI) that implements this specification. It is in development in the [devcontainers/cli](https://github.com/devcontainers/cli) repo.

### Remote - Containers CLI

There is a Remote - Containers [`devcontainer` CLI](https://code.visualstudio.com/docs/remote/devcontainer-cli) which may be installed within Remote - Containers or through the command line.

### Visual Studio Code Remote - Containers

The [**Visual Studio Code Remote - Containers** extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers) lets you use a container as a full-featured development environment. It allows you to open any folder inside (or mounted into) a container and take advantage of Visual Studio Code's full feature set. There is more information in the Remote - Containers [documentation](https://code.visualstudio.com/docs/remote/containers).

> **Tip:** If you've already built a container and connected to it, be sure to run **Remote-Containers: Rebuild Container** from the Command Palette (`kbstyle(F1)`) to pick up any changes you make.

#### Product specific properties

Remote containers implements the [VS Code properties](#visual-studio-code) specific properties.

#### Product specific limitations

Some properties may also have certain limitations in the Remote - Containers extension.

| Property or variable | Type | Description |
|----------|------|-------------|
| `workspaceMount` | string | Not yet supported when using Clone Repository in Container Volume. |
| `workspaceFolder` | string | Not yet supported when using Clone Repository in Container Volume. |
| `${localWorkspaceFolder}`  | Any | Not yet supported when using Clone Repository in Container Volume. |
| `${localWorkspaceFolderBasename}` | Any | Not yet supported when using Clone Repository in Container Volume. |

### GitHub Codespaces

A [codespace](https://docs.github.com/en/codespaces/overview) is a development environment that's hosted in the cloud. Codespaces run on a variety of VM-based compute options hosted by GitHub.com, which you can configure from 2 core machines up to 32 core machines. You can connect to your codespaces from the browser or locally using Visual Studio Code.

> **Tip:** If you've already built a codespace and connected to it, be sure to run **Codespaces: Rebuild Container** from the Command Palette (`kbstyle(F1)`) to pick up any changes you make.

> **Tip** Codespaces implements an auto `workspaceFolder` mount in **Docker Compose** scenarios.

#### Product specific properties
GitHub Codespaces works with a growing number of tools and, where applicable, their `devcontainer.json` properties. For example, connecting the Codespaces web editor or VS Code enables the use of [VS Code properties](#visual-studio-code).

If your Codespaces project needs additional permissions for other repositories, you can configure this through the `repositories` and `permissions` properties. You may learn more about this in the [Codespaces documentation](https://docs.github.com/en/codespaces/managing-your-codespaces/managing-repository-access-for-your-codespaces). As with other tools, Codespaces specific properties are placed within a `codespaces` namespace inside the `customizations` property.

```jsonc
"customizations": {
		// Configure properties specific to Codespaces.
		"codespaces": {
			"repositories": {
				"my_org/my_repo": {
					"permissions": {
						"issues": "write",
						"contents": "read"
					}
				}
			}
		}
}
```

#### Product specific limitations

Some properties may apply differently to Codespaces.

| Property or variable | Type | Description |
|----------|---------|----------------------|
| `mounts` | array | Codespaces ignores "bind" mounts with the exception of the Docker socket. Volume mounts are still allowed.|
| `workspaceMount` | string | Not yet supported in Codespaces. |
| `workspaceFolder` | string | Not yet supported in Codespaces. |
| `forwardPorts` | array | Codespaces does not yet support the `"host:port"` variation of this property. |
| `portsAttributes` | object | Codespaces does not yet support the `"host:port"` variation of this property.|
| `shutdownAction` | enum | Does not apply to Codespaces. |
| `${localEnv:VARIABLE_NAME}` | Any | For Codespaces, the host is in the cloud rather than your local machine.|
