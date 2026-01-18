# Supporting Tools and Services

**Note: For the latest set of supporting tools, please check out https://containers.dev/supporting.**

This page outlines tools and services that currently support the Development Container Specification, including the `devcontainer.json` format. A `devcontainer.json` file in your project tells tools and services that support the dev container spec how to access (or create) a dev container with a well-defined tool and runtime stack. 

While most [dev container properties](devcontainerjson-reference.md) apply to any supporting tool or service, a few are specific to certain tools, which are outlined below.

## Editors

### Visual Studio Code

Visual Studio Code specific properties go under `vscode` inside `customizations`.


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

Please note that the VS Code [Dev Containers](#visual-studio-code-remote---containers) extension and [GitHub Codespaces](#github-codespaces) support the VS Code properties.

### Visual Studio

Visual Studio added dev container support in Visual Studio 2022 17.4 for C++ projects using CMake Presets. It is part of the Linux and embedded development with C++ workload, so make sure it is selected in your VS installation. Visual Studio manages the lifecycle of dev containers it uses as you work, but it treats them as remote targets in a similar way to other Linux or WSL targets.

You may learn more in the [announcement blog post](https://devblogs.microsoft.com/cppblog/dev-containers-for-c-in-visual-studio/).

### Zed

[Zed v0.218](https://zed.dev/releases/stable) and onward supports Dev Containers, but it is currently still in development with the following limitations:

* Extensions: Zed does not yet manage extensions separately for container environments. The host's extensions are used as-is.
* Port forwarding: Only the appPort field is supported. forwardPorts and other advanced port-forwarding features are not implemented.
* Configuration changes: Updates to devcontainer.json do not trigger automatic rebuilds or reloads; containers must be manually restarted.

For more information please see Zed's up-to-date [Dev Containers documentation](https://zed.dev/docs/dev-containers) and their [announcement blog post](https://zed.dev/blog/dev-containers).


## Tools

### Dev Container CLI

A dev container command line interface (CLI) that implements this specification. It is in development in the [devcontainers/cli](https://github.com/devcontainers/cli) repo.

### VS Code extension CLI

VS Code has a [CLI](https://code.visualstudio.com/docs/remote/devcontainer-cli) which may be installed within the Dev Containers extension or through the command line.

### Cachix devenv

Cachix's [devenv](https://devenv.sh/) supports automatically generating a `.devcontainer.json` file so you can use it with any Dev Container Spec supporting tool. See [devenv documentation](https://devenv.sh/integrations/codespaces-devcontainer/) for detais. 

### Jetpack.io Devbox

[Jetpack.io's VS Code extension](https://marketplace.visualstudio.com/items?itemName=jetpack-io.devbox) supports a **Generate Dev Container files** command so you can use Jetpack.io from Dev Container Spec supporting tools.

### Visual Studio Code Dev Containers

The [**Visual Studio Code Dev Containers** extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers) lets you use a container as a full-featured development environment. It allows you to open any folder inside (or mounted into) a container and take advantage of Visual Studio Code's full feature set. There is more information in the Dev Containers [documentation](https://code.visualstudio.com/docs/remote/containers).

> **Tip:** If you've already built a container and connected to it, be sure to run **Dev Containers: Rebuild Container** from the Command Palette (`kbstyle(F1)`) to pick up any changes you make.

#### Product specific properties

Dev Containers implements the [VS Code](#visual-studio-code) specific properties.

#### Product specific limitations

Some properties may also have certain limitations in the Dev Containers extension.

| Property or variable | Type | Description |
|----------|------|-------------|
| `workspaceMount` | string | Not yet supported when using Clone Repository in Container Volume. |
| `workspaceFolder` | string | Not yet supported when using Clone Repository in Container Volume. |
| `${localWorkspaceFolder}`  | Any | Not yet supported when using Clone Repository in Container Volume. |
| `${localWorkspaceFolderBasename}` | Any | Not yet supported when using Clone Repository in Container Volume. |

### GitHub Codespaces

A [codespace](https://docs.github.com/en/codespaces/overview) is a development environment that's hosted in the cloud. Codespaces run on a variety of VM-based compute options hosted by GitHub.com, which you can configure from 2 core machines up to 32 core machines. You can connect to your codespaces from the browser or locally using Visual Studio Code.

> **Tip:** If you've already built a codespace and connected to it, be sure to run **Codespaces: Rebuild Container** from the Command Palette (`kbstyle(F1)`) to pick up any changes you make.

> **Tip:** Codespaces implements an auto `workspaceFolder` mount in **Docker Compose** scenarios.

#### Product specific properties
GitHub Codespaces works with a growing number of tools and, where applicable, their `devcontainer.json` properties. For example, connecting the codespaces web editor or VS Code enables the use of [VS Code properties](#visual-studio-code).

If your codespaces project needs additional permissions for other repositories, you can configure this through the `repositories` and `permissions` properties. You may learn more about this in the [Codespaces documentation](https://docs.github.com/en/codespaces/managing-your-codespaces/managing-repository-access-for-your-codespaces). As with other tools, Codespaces specific properties are placed within a `codespaces` namespace inside the `customizations` property.

```jsonc
"customizations": {
	// Configure properties specific to Codespaces.
	"codespaces": {
		"repositories": {
			"my_org/my_repo": {
				"permissions": {
					"issues": "write"
				}
			}
		}
	}
}
```

You can customize which files are initially opened when the codespace is created:
```jsonc
"customizations": {
	// Configure properties specific to Codespaces.
	"codespaces": {
		"openFiles": [
			"README"
			"src/index.js"
		]
	}
}
```

The paths are relative to the root of the repository. They will be opened in order, with the first file activated.

Codespaces will automatically perform some default setup when the `devcontainer.json` does not specify a `postCreateCommand`. This can be disabled with the `disableAutomaticConfiguration` setting:

```jsonc
"customizations": {
	// Configure properties specific to Codespaces.
	"codespaces": {
		"disableAutomaticConfiguration": true
	}
}
```

Note that currently codespaces reads these properties from `devcontainer.json`, not image metadata.

#### Product specific limitations

Some properties may apply differently to codespaces.

| Property or variable | Type | Description |
|----------|---------|----------------------|
| `mounts` | array | Codespaces ignores "bind" mounts with the exception of the Docker socket. Volume mounts are still allowed.|
| `forwardPorts` | array | Codespaces does not yet support the `"host:port"` variation of this property. |
| `portsAttributes` | object | Codespaces does not yet support the `"host:port"` variation of this property.|
| `shutdownAction` | enum | Does not apply to Codespaces. |
| `${localEnv:VARIABLE_NAME}` | Any | For Codespaces, the host is in the cloud rather than your local machine.|
| `customizations.codespaces` | object | Codespaces reads this property from devcontainer.json, not image metadata. |
| `hostRequirements` | object | Codespaces reads this property from devcontainer.json, not image metadata. |
