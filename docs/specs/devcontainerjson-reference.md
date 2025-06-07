# Dev Container Metadata Reference

The `devcontainer.json` file contains any needed metadata and settings required to configurate a **development container** for a given well-defined tool and runtime stack. It can be used by [tools and services that support the dev container spec](supporting-tools.md) to create a **development environment** that contains one or more **development containers**.

Metadata properties marked with a 🏷️ can be stored in the `devcontainer.metadata` **container image label** in addition to `devcontainer.json`. This label can contain an array of json snippets that will be automatically merged with `devcontainer.json` contents (if any) when a container is created.

## General devcontainer.json properties

| Property | Type | Description |
|----------|------|-------------|
| `name` | string | A name for the dev container displayed in the UI |
| `forwardPorts` 🏷️ | array | An array of port numbers or `"host:port"` values  (e.g. `[3000, "db:5432"]`) that should always be forwarded from inside the primary container to the local machine (including on the web). The property is most useful for forwarding ports that cannot be auto-forwarded because the related process that starts before the `devcontainer.json` supporting service / tool connects or for forwarding a service not in the primary container in Docker Compose scenarios (e.g. `"db:5432"`). Defaults to `[]`. |
| `portsAttributes` 🏷️ | object | Object that maps a port number, `"host:port"` value, range, or regular expression to a set of default options. See [port attributes](#port-attributes) for available options. For example: <br />`"portsAttributes": {"3000": {"label": "Application port"}}` |
| `otherPortsAttributes` 🏷️ | object | Default options for ports, port ranges, and hosts that aren't configured using `portsAttributes`. See [port attributes](#port-attributes) for available options. For example: <br /> `"otherPortsAttributes": {"onAutoForward": "silent"}` |
| `containerEnv` 🏷️ | object | A set of name-value pairs that sets or overrides environment variables for the container. Environment and [pre-defined variables](#variables-in-devcontainerjson) may be referenced in the values. For example:<br/> `"containerEnv": { "MY_VARIABLE": "${localEnv:MY_VARIABLE}" }`<br /> If you want to reference an existing container variable while setting this one (like updating the `PATH`), use `remoteEnv` instead. <br /> `containerEnv` will set the variable on the Docker container itself, so all processes spawned in the container will have access to it. But it will also be static for the life of the container - you must rebuild the container to update the value. <br /> We recommend using `containerEnv` (over `remoteEnv`) as much as possible since it allows all processes to see the variable and isn't client-specific. |
| `remoteEnv` 🏷️ | object | A set of name-value pairs that sets or overrides environment variables for the `devcontainer.json` supporting service / tool (or sub-processes like terminals) but not the container as a whole. Environment and [pre-defined variables](#variables-in-devcontainerjson) may be referenced in the values. <br /> You may want to use `remoteEnv` (over `containerEnv`) if the value isn't static since you can update its value without having to rebuild the full container. |
| `remoteUser` 🏷️ | string | Overrides the user that `devcontainer.json` supporting services tools / runs as in the container (along with sub-processes like terminals, tasks, or debugging). Does not change the user the container as a whole runs as which can be set using `containerUser`. Defaults to the user the container as a whole is running as (often `root`). <br /> You may learn more in the [remoteUser section below](#remoteUser). |
| `containerUser` 🏷️ | string | Overrides the user for all operations run as inside the container. Defaults to either `root` or the last `USER` instruction in the related Dockerfile used to create the image. If you want any connected tools or related processes to use a different user than the one for the container, see `remoteUser`. |
| `updateRemoteUserUID` 🏷️ | boolean | On Linux, if `containerUser` or `remoteUser` is specified, the user's UID/GID will be updated to match the local user's UID/GID to avoid permission problems with bind mounts. Defaults to `true`. |
| `userEnvProbe` 🏷️ | enum | Indicates the type of shell to use to "probe" for user environment variables to include in `devcontainer.json` supporting services' / tools' processes: `none`, `interactiveShell`, `loginShell`, or `loginInteractiveShell` (default). The specific shell used is based on the default shell for the user (typically bash). For example, bash interactive shells will typically include variables set in `/etc/bash.bashrc` and `~/.bashrc` while login shells usually include variables from `/etc/profile` and `~/.profile`. Setting this property to `loginInteractiveShell` will get variables from all four files. |
| `overrideCommand` 🏷️ | boolean | Tells `devcontainer.json` supporting services / tools whether they should run `/bin/sh -c "while sleep 1000; do :; done"` when starting the container instead of the container's default command (since the container can shut down if the default command fails). Set to `false` if the default command must run for the container to function properly. Defaults to `true` for when using an image Dockerfile and `false` when referencing a Docker Compose file. |
| `shutdownAction` 🏷️ | enum | Indicates whether `devcontainer.json` supporting tools should stop the containers when the related tool window is closed / shut down.<br>Values are  `none`, `stopContainer` (default for image or Dockerfile), and `stopCompose` (default for Docker Compose). |
| `init` 🏷️ | boolean | Defaults to `false`. Cross-orchestrator way to indicate whether the [tini init process](https://github.com/krallin/tini) should be used to help deal with zombie processes. |
| `privileged` 🏷️ | boolean | Defaults to `false`. Cross-orchestrator way to cause the container to run in privileged mode (`--privileged`). Required for things like Docker-in-Docker, but has security implications particularly when running directly on Linux.  |
| `capAdd` 🏷️ | array | Defaults to `[]`. Cross-orchestrator way to add capabilities typically disabled for a container. Most often used to add the `ptrace` capability required to debug languages like C++, Go, and Rust. For example: <br />`"capAdd": ["SYS_PTRACE"]` |
| `securityOpt` 🏷️ | array | Defaults to `[]`. Cross-orchestrator way to set container security options. For example: <br /> `"securityOpt": [ "seccomp=unconfined" ]` |
| `mounts` 🏷️ | string or object | Defaults to unset. Cross-orchestrator way to add additional mounts to a container. Each value is a string that accepts the same values as the [Docker CLI `--mount` flag](https://docs.docker.com/engine/reference/commandline/run/#mount). Environment and [pre-defined variables](#variables-in-devcontainerjson) may be referenced in the value. For example:<br />`"mounts": [{ "source": "dind-var-lib-docker", "target": "/var/lib/docker", "type": "volume" }]` |
| `features` | object | An object of [Dev Container Feature IDs](https://containers.dev/features) and related options to be added into your primary container. The specific options that are available varies by feature, so see its documentation for additional details. For example: <br />`"features": { "ghcr.io/devcontainers/features/github-cli": {} }` |
| `overrideFeatureInstallOrder` | array | By default, Features will attempt to automatically set the order they are installed based on a `installsAfter` property within each of them. This property allows you to override the Feature install order when needed. For example: <br />`"overrideFeatureInstallОrder": [ "ghcr.io/devcontainers/features/common-utils", "ghcr.io/devcontainers/features/github-cli" ]` |
| `customizations` 🏷️| object | Product specific properties, defined in [supporting tools](supporting-tools.md) |

## Scenario specific properties

The focus of `devcontainer.json` is to describe how to enrich a container for the purposes of development rather than acting as a multi-container orchestrator format. Instead, container orchestrator formats can be referenced when needed to manage multiple containers and their lifecycles. Today, `devcontainer.json` includes scenario specific properties for working without a container orchestrator (by directly referencing an image or Dockerfile) and for using Docker Compose as a simple multi-container orchestrator.

### Image or Dockerfile specific properties

| Property | Type | Description |
|----------|------|-------------|
| `image` | string | **Required** when using an image. The name of an image in a container registry ([DockerHub](https://hub.docker.com), [GitHub Container Registry](https://docs.github.com/packages/guides/about-github-container-registry), [Azure Container Registry](https://azure.microsoft.com/services/container-registry/)) that `devcontainer.json` supporting services / tools should use to create the dev container. |
| `build.dockerfile` | string |**Required** when using a Dockerfile. The location of a [Dockerfile](https://docs.docker.com/engine/reference/builder/) that defines the contents of the container. The path is relative to the `devcontainer.json` file. |
| `build.context` | string | Path that the Docker build should be run from relative to `devcontainer.json`. For example, a value of `".."` would allow you to reference content in sibling directories. Defaults to `"."`. |
| `build.args` | Object | A set of name-value pairs containing [Docker image build arguments](https://docs.docker.com/engine/reference/commandline/build/#set-build-time-variables---build-arg) that should be passed when building a Dockerfile.  Environment and [pre-defined variables](#variables-in-devcontainerjson) may be referenced in the values. Defaults to not set. For example: `"build": { "args": { "MYARG": "MYVALUE", "MYARGFROMENVVAR": "${localEnv:VARIABLE_NAME}" } }` |
| `build.options` | array | An array of [Docker image build options](https://docs.docker.com/engine/reference/commandline/build/#options) that should be passed to the build command when building a Dockerfile. Defaults to `[]`. For example: `"build": { "options": [ "--add-host=host.docker.internal:host-gateway" ] }` |
| `build.target` | string | A string that specifies a [Docker image build target](https://docs.docker.com/engine/reference/commandline/build/#specifying-target-build-stage---target) that should be passed when building a Dockerfile. Defaults to not set. For example: `"build": { "target": "development" }` |
| `build.cacheFrom` | string,<br>array | A string or array of strings that specify one or more images to use as caches when building the image. Cached image identifiers are passed to the `docker build` command with `--cache-from`. |
| `appPort` | integer,<br>string,<br>array |  In most cases, we recommend using the new [forwardPorts property](#general-devcontainerjson-properties). This property accepts a port or array of ports that should be published locally when the container is running. Unlike `forwardPorts`, your application may need to listen on all interfaces (`0.0.0.0`) not just `localhost` for it to be available externally. Defaults to `[]`.<br>Learn more about publishing vs forwarding ports [here](#publishing-vs-forwarding-ports).<br>Note that the array syntax will execute the command without a shell. You can [learn more](#formatting-string-vs-array-properties) about formatting string vs array properties. |
| `workspaceMount` | string | Requires `workspaceFolder` be set as well. Overrides the default local mount point for the workspace when the container is created. Supports the same values as the [Docker CLI `--mount` flag](https://docs.docker.com/engine/reference/commandline/run/#add-bind-mounts-or-volumes-using-the---mount-flag). Environment and [pre-defined variables](#variables-in-devcontainerjson) may be referenced in the value. For example: <br />`"workspaceMount": "source=${localWorkspaceFolder}/sub-folder,target=/workspace,type=bind,consistency=cached", "workspaceFolder": "/workspace"` |
| `workspaceFolder` | string | Requires `workspaceMount` be set. Sets the default path that `devcontainer.json` supporting services / tools should open when connecting to the container. Defaults to the automatic source code mount location. |
| `runArgs` | array | An array of [Docker CLI arguments](https://docs.docker.com/engine/reference/commandline/run/) that should be used when running the container. Defaults to `[]`. For example, this allows ptrace based debuggers like C++ to work in the container:<br /> `"runArgs": [ "--cap-add=SYS_PTRACE", "--security-opt", "seccomp=unconfined" ]` . |

### Docker Compose specific properties

| Property | Type | Description |
|----------|------|-------------|
| `dockerComposeFile` | string,<br>array | **Required** when [using Docker Compose](https://docs.docker.com/compose/). Path or an ordered list of paths to Docker Compose files relative to the `devcontainer.json` file. Using an array is useful [when extending your Docker Compose configuration](https://docs.docker.com/compose/extends/#multiple-compose-files). The order of the array matters since the contents of later files can override values set in previous ones.<br>The default `.env` file is picked up from the root of the project, but you can use `env_file` in your Docker Compose file to specify an alternate location. |
| `service` | string | **Required** when [using Docker Compose](https://docs.docker.com/compose/). The name of the service `devcontainer.json` supporting services / tools should connect to once running.  |
| `runServices` | array | An array of services in your Docker Compose configuration that should be started by `devcontainer.json` supporting services / tools. These will also be stopped when you disconnect unless `"shutdownAction"` is `"none"`. Defaults to all services. |
| `workspaceFolder` | string | Sets the default path that `devcontainer.json` supporting services / tools should open when connecting to the container (which is often the path to a volume mount where the source code can be found in the container). Defaults to `"/"`. |

## Tool-specific properties

While most properties apply to any `devcontainer.json` supporting tool or service, a few are specific to certain tools. You may explore this in the [supporting tools and services document](supporting-tools.md).

## Lifecycle scripts

When creating or working with a dev container, you may need different commands to be run at different points in the container's lifecycle. The table below lists a set of command properties you can use to update what the container's contents in the order in which they are run (for example, `onCreateCommand` will run after `initializeCommand`). Each command property is an string or list of command arguments that should execute from the `workspaceFolder`.

| Property | Type | Description |
|----------|------|-------------|
| `initializeCommand` | string,<br>array,<br>object | A command string or list of command arguments to run on the **host machine** during initialization, including during container creation and on subsequent starts.  The command may run more than once during a given session.<br /><br /> ⚠️ The command is run wherever the source code is located on the host. For cloud services, this is in the cloud.<br>Note that the array syntax will execute the command without a shell. You can [learn more](#formatting-string-vs-array-properties) about formatting string vs array vs object properties. |
| `onCreateCommand` 🏷️ | string,<br>array,<br>object | This command is the first of three (along with `updateContentCommand` and `postCreateCommand`) that finalizes container setup when a dev container is created. It and subsequent commands execute **inside** the container immediately after it has started for the first time.<br /><br> Cloud services can use this command when caching or prebuilding a container. This means that it will not typically have access to user-scoped assets or secrets.<br>Note that the array syntax will execute the command without a shell. You can [learn more](#formatting-string-vs-array-properties) about formatting string vs array vs object properties. |
| `updateContentCommand` 🏷️ | string,<br>array,<br>object  | This command is the second of three that finalizes container setup when a dev container is created. It executes inside the container after `onCreateCommand` whenever new content is available in the source tree during the creation process.<br><br />It will execute at least once, but cloud services will also periodically execute the command to refresh cached or prebuilt containers. Like cloud services using `onCreateCommand`, it can only take advantage of repository and org scoped secrets or permissions.<br>Note that the array syntax will execute the command without a shell. You can [learn more](#formatting-string-vs-array-properties) about formatting string vs array vs object properties. |
| `postCreateCommand` 🏷️ | string,<br>array,<br>object | This command is the last of three that finalizes container setup when a dev container is created. It happens after `updateContentCommand` and once the dev container has been assigned to a user for the first time.<br><br />Cloud services can use this command to take advantage of user specific secrets and permissions.<br>Note that the array syntax will execute the command without a shell. You can [learn more](#formatting-string-vs-array-properties) about formatting string vs array vs object properties. |
| `postStartCommand` 🏷️ | string,<br>array,<br>object | A command to run each time the container is successfully started.<br>Note that the array syntax will execute the command without a shell. You can [learn more](#formatting-string-vs-array-properties) about formatting string vs array vs object properties. |
| `postAttachCommand` 🏷️ | string,<br>array,<br>object | A command to run each time a tool has successfully attached to the container.<br>Note that the array syntax will execute the command without a shell. You can [learn more](#formatting-string-vs-array-properties) about formatting string vs array vs object properties. |
| `waitFor` 🏷️ | enum | An enum that specifies the command any tool should wait for before connecting. Defaults to `updateContentCommand`. This allows you to use `onCreateCommand` or `updateContentCommand` for steps that must happen before `devcontainer.json` supporting tools connect while still using `postCreateCommand` for steps that can happen behind the scenes afterwards. |

For each command property, if the value is a single string, it will be run in `/bin/sh`. Use `&&` in a string to execute multiple commands. For example, `"yarn install"` or `"apt-get update && apt-get install -y curl"`. The array syntax `["yarn", "install"]` will invoke the command (in this case `yarn`) directly without using a shell. Each fires after your source code has been mounted, so you can also run shell scripts from your source tree. For example: `bash scripts/install-dev-tools.sh`

If one of the lifecycle scripts fails, any subsequent scripts will not be executed. For instance, if `postCreateCommand` fails, `postStartCommand` and any following scripts will be skipped.

## Minimum host requirements

While `devcontainer.json` does not focus on hardware or VM provisioning, it can be useful to know your container's minimum RAM, CPU, and storage requirements. This is what the `hostRequirements` properties allow you to do. Cloud services can use these properties to automatically default to the best compute option available, while in other cases, you will be presented with a warning if the requirements are not met.

| Property | Type | Description |
|----------|------|-------------|
| `hostRequirements.cpus` 🏷️ | integer | Indicates the minimum required number of CPUs / virtual CPUs / cores. For example: `"hostRequirements": {"cpus": 2}` |
| `hostRequirements.memory` 🏷️ | string |  A string indicating minimum memory requirements with a `tb`, `gb`, `mb`, or `kb` suffix. For example, `"hostRequirements": {"memory": "4gb"}` |
| `hostRequirements.storage` 🏷️ | string | A string indicating minimum storage requirements with a `tb`, `gb`, `mb`, or `kb` suffix. For example, `"hostRequirements": {"storage": "32gb"}` |

## Port attributes

The `portsAttributes` and `otherPortsAttributes` properties allow you to map default port options for one or more manually or automatically forwarded ports. The following is a list of options that can be set in the configuration object assigned to the property.

| Property | Type | Description |
|----------|------|-------------|
| `label` 🏷️ | string | Display name for the port in the ports view. Defaults to not set. |
| `protocol` 🏷️ | enum | Controls protocol handling for forwarded ports. When not set, the port is assumed to be a raw TCP stream which, if forwarded to `localhost`, supports any number of protocols. However, if the port is forwarded to a web URL (e.g. from a cloud service on the web), only HTTP ports in the container are supported. Setting this property to `https` alters handling by ignoring any SSL/TLS certificates present when communicating on the port and using the correct certificate for the forwarded URL instead (e.g `https://*.githubpreview.dev`). If set to `http`, processing is the same as if the protocol is not set. Defaults to not set. |
| `onAutoForward` 🏷️ | enum | Controls what should happen when a port is auto-forwarded once you've connected to the container. `notify` is the default, and a notification will appear when the port is auto-forwarded. If set to `openBrowser`, the port will be opened in the system's default browser. A value of `openBrowserOnce` will open the browser only once. `openPreview` will open the URL in `devcontainer.json` supporting services' / tools' embedded preview browser. A value of `silent` will forward the port, but take no further action. A value of `ignore` means that this port should not be auto-forwarded at all. |
| `requireLocalPort` 🏷️ | boolean | Dictates when port forwarding is required to map the port in the container to the same port locally or not. If set to `false`, the `devcontainer.json` supporting services /  tools will attempt to use the specified port forward to `localhost`, and silently map to a different one if it is unavailable. If set to `true`, you will be notified if it is not possible to use the same port. Defaults to `false`. |
| `elevateIfNeeded` 🏷️ | boolean | Forwarding low ports like 22, 80, or 443 to `localhost` on the same port from `devcontainer.json` supporting services / tools may require elevated permissions on certain operating systems. Setting this property to `true` will automatically try to elevate the `devcontainer.json` supporting tool's permissions in this situation. Defaults to `false`. |

## Formatting string vs. array properties

The format of certain properties will vary depending on the involvement of a shell.

`postCreateCommand`, `postStartCommand`, `postAttachCommand`, and `initializeCommand` all have 3 types: 
* Array: Passed to the OS for execution without going through a shell
* String: Goes through a shell (it needs to be parsed into command and arguments)
* Object: All lifecycle scripts have been extended to support `object` types to allow for [parallel execution](../specs/devcontainer-reference.md/#parallel-lifecycle-script-execution)

`runArgs` only has the array type. Using `runArgs` via a typical command line, you'll need single quotes if the shell runs into parameters with spaces. However, these single quotes aren't passed on to the executable. Thus, in your `devcontainer.json`, you'd follow the array format and leave out the single quotes:

```json
"runArgs": ["--device-cgroup-rule=my rule here"]
```

Rather than:

```json
"runArgs": ["--device-cgroup-rule='my rule here'"]
```

We can compare the string, array, and object versions of `postAttachCommand` as well. You can use the following string format, which will remove the single quotes as part of the shell's parsing:

```json
"postAttachCommand": "echo foo='bar'"
```

By contrast, the array format will keep the single quotes and write them to standard out (you can see the output in the dev container log):

```json
"postAttachCommand": ["echo", "foo='bar'"]
```

Finally, you may use an object format:

```json
{
  "postAttachCommand": {
    "server": "npm start",
    "db": ["mysql", "-u", "root", "-p", "my database"]
  }
}
```

## Variables in devcontainer.json

Variables can be referenced in certain string values in `devcontainer.json` in the following format: **${variableName}**. The following is a list of available variables you can use.

| Variable | Properties | Description |
|----------|---------|----------------------|
| `${localEnv:VARIABLE_NAME}` | Any | Value of an environment variable on the **host machine** (in the examples below, called `VARIABLE_NAME`). Unset variables are left blank. <br /><br />  ⚠️ Clients (like VS Code) may need to be **restarted** to pick up newly set variables. <br /><br /> ⚠️ For a cloud service, the host is in the cloud rather than your local machine. <br /><br /> **Examples** <br /><br /> **1.** Set a variable containing your local home folder on Linux / macOS or the user folder on Windows:<br/> `"remoteEnv": { "LOCAL_USER_PATH": "${localEnv:HOME}${localEnv:USERPROFILE}" }`.   <br /><br /> A default value for when the environment variable is not set can be given with `${localEnv:VARIABLE_NAME:default_value}`.  <br /><br />  **2.** In modern versions of macOS, default configurations allow setting local variables with the command `echo 'export VARIABLE_NAME=my-value' >> ~/.zshenv`.  |
| `${containerEnv:VARIABLE_NAME}` | `remoteEnv` | Value of an existing environment variable inside the container once it is up and running (in this case, called `VARIABLE_NAME`). For example:<br /> `"remoteEnv": { "PATH": "${containerEnv:PATH}:/some/other/path" }` <br /><br /> A default value for when the environment variable is not set can be given with `${containerEnv:VARIABLE_NAME:default_value}`. |
| `${localWorkspaceFolder}`  | Any | Path of the local folder that was opened in the `devcontainer.json` supporting service / tool (that contains `.devcontainer/devcontainer.json`). |
| `${containerWorkspaceFolder}` | Any | The path that the workspaces files can be found in the container. |
| `${localWorkspaceFolderBasename}` | Any | Name of the local folder that was opened in the `devcontainer.json` supporting service / tool (that contains `.devcontainer/devcontainer.json`). |
| `${containerWorkspaceFolderBasename}` | Any | Name of the folder where the workspace files can be found in the container. |
| `${devcontainerId}` | Any | Allow features to refer to an identifier that is unique to the dev container they are installed into and that is stable across rebuilds.<br> The properties supporting it in devcontainer.json are: `name`, `runArgs`, `initializeCommand`, `onCreateCommand`, `updateContentCommand`, `postCreateCommand`, `postStartCommand`, `postAttachCommand`, `workspaceFolder`, `workspaceMount`, `mounts`, `containerEnv`, `remoteEnv`, `containerUser`, `remoteUser`, and `customizations`. |

## Schema

You can see the dev container schema [here](https://github.com/devcontainers/spec/blob/main/schemas/devContainer.base.schema.json).


## Publishing vs forwarding ports

Docker has the concept of "publishing" ports when the container is created. Published ports behave very much like ports you make available to your local network. If your application only accepts calls from `localhost`, it will reject connections from published ports just as your local machine would for network calls. Forwarded ports, on the other hand, actually look like `localhost` to the application.

## remoteUser

A dev container configuration will inherit the `remoteUser` property from the base image it uses.

Using the [images](https://github.com/devcontainers/images) and [Templates](https://github.com/devcontainers/templates) part of the spec as an example: `remoteUser` in these images is set to a custom value - you may view an example in the [C++ image](https://github.com/devcontainers/images/blob/main/src/cpp/.devcontainer/devcontainer.json#L34). The [C++ Template](https://github.com/devcontainers/templates/tree/main/src/cpp) will then inherit the custom `remoteUser` value from [its base C++ image](https://github.com/devcontainers/templates/blob/main/src/cpp/.devcontainer/Dockerfile#L1).
