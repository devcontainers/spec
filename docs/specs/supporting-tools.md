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

### IntelliJ IDEA

IntelliJ IDEA supports dev containers that can be run remotely via an SSH connection or locally using Docker.

You may learn more in the [IntelliJ IDEA documentation](https://www.jetbrains.com/help/idea/connect-to-devcontainer.html).

### Emacs

[GNU Emacs](https://www.gnu.org/software/emacs/) can make use of dev containers using the community extension [devcontainer.el](https://johannes-mueller.github.io/devcontainer.el/).  The extension is available by the popular package registry [MELPA](https://melpa.org).  It can also be installed directly from the source code that is available on [GitHub](https://github.com/johannes-mueller/devcontainer.el).

## Tools

### Dev Container CLI

A dev container command line interface (CLI) that implements this specification. It is in development in the [devcontainers/cli](https://github.com/devcontainers/cli) repo.

### VS Code extension CLI

VS Code has a [CLI](https://code.visualstudio.com/docs/remote/devcontainer-cli) which may be installed within the Dev Containers extension or through the command line.

### Cachix devenv

Cachix's [devenv](https://devenv.sh/) supports automatically generating a `.devcontainer.json` file so you can use it with any Dev Container Spec supporting tool. See [devenv documentation](https://devenv.sh/integrations/codespaces-devcontainer/) for detais. 

### Jetify Devbox

[Jetify](https://jetify.com) (formerly jetpack.io) is a [Nix](https://nixos.org/)-based service for deploying applications. [DevBox](https://www.jetify.com/devbox/) provides a way to use Nix to generate a development environment. [Jetify's VS Code extension](https://marketplace.visualstudio.com/items?itemName=jetpack-io.devbox) allows you to quickly take advantage of DevBox in any Dev Container Spec supporting tool or service.

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

### CodeSandbox

[CodeSandbox](https://codesandbox.io/) provides cloud development environments running on a microVM architecture.

When you import a GitHub repository into CodeSandbox, it will automatically provision a dedicated environment for every branch. Thanks to memory snapshotting, CodeSandbox then resumes and branches an environment in under two seconds.

CodeSandbox offers support for multiple editors, so you can code using the CodeSandbox web editor, VS Code, or the CodeSandbox iOS app.

### DevPod

[DevPod](https://github.com/loft-sh/devpod) is a client-only tool to create reproducible developer environments based on a `devcontainer.json` on any backend. Each developer environment runs in a container and is specified through a `devcontainer.json`. Through DevPod providers these environments can be created on any backend, such as the local computer, a Kubernetes cluster, any reachable remote machine or in a VM in the cloud.

### Ona (formerly Gitpod)

[Ona](https://ona.com/) (formerly Gitpod) is the mission control for software projects and software engineering agents. It provides secure, ephemeral development environments that run in our cloud or your VPC, enabling humans and agents to collaborate seamlessly.

For details on constraints, customization, and automation options, see the [Ona Dev Container docs](https://ona.com/docs/ona/configuration/devcontainer/overview).
