# [Proposal] Allow Dev Container Features to contribute lifecycle scripts

Related to: https://github.com/devcontainers/spec/issues/60, https://github.com/devcontainers/spec/issues/181

## Goal

Allow Feature authors to provide lifecycle hooks for their Features during a dev container build.

## Proposal

 Introduce the following properties to [devcontainer-feature.json](https://containers.dev/implementors/features/#devcontainer-feature-json-properties), mirroring the behavior and syntax of the `devcontainer.json` lifecycle hooks.

- "onCreateCommand"
- "updateContentCommand"
- "postCreateCommand"
- "postStartCommand"
- "postAttachCommand"

Note that `initializeCommand` is omitted, pending further discussions around a secure design.

Additionally, introduce a `${featureRoot}` dev container variable, which is expanded at build time to the root of the Feature the variable is referenced from.  

As with all lifecycle hooks, commands are executed from the context (cwd) of the [project workspace folder](https://containers.dev/implementors/spec/#project-workspace-folder).

All other semantic match the existing [Lifecycle Scripts](https://containers.dev/implementors/json_reference/#lifecycle-scripts) and  [lifecycle script parallel execution](https://containers.dev/implementors/spec/#parallel-exec) behavior exactly.

###  Execution

When a dev container is brought up, for each lifecycle hook, each Feature that contributes a command to a lifecycle hook shall have the command executed in sequence, following the same execution order as outlined in [Feature installation order](https://containers.dev/implementors/features/#installation-order), and always before any user-provided lifecycle commands.

## Examples


### Executing a script in the workspace folder

The follow example illustrates contributing an `onCreateCommand` and `postCreateCommand` script to `featureA`.  At each lifecycle hook during the build, the provided script will be executed in the [project workspace folder](https://containers.dev/implementors/spec/#project-workspace-folder), following the same semantics as [user-defined lifecycle hooks](https://containers.dev/implementors/json_reference/#lifecycle-scripts).

```jsonc
{
   "id": "featureA",
   "version": "1.0.0",
   "onCreateCommand": "myOnCreate.sh && myOnCreate2.sh",
   "postCreateCommand": "myPostCreate.sh",
    "postAttachCommand": {
        "command01": "myPostAttach.sh arg01",
        "command02": "myPostAttach.sh arg02",
    },
}

```

### Executing a script bundled with the Feature

The following example illustrates executing a `postCreateCommand` script bundled with the Feature utilizing the `${featureRoot}` variable.

```jsonc
{
   "id": "featureB",
   "version": "1.0.0",
   "postCreateCommand": "${featureRoot}/bundledScript.sh",
   "installsAfter": [
        "featureA"
   ]
}
```

At build time, the `${featureRoot}` variable will be expanded to the temporary directory within the container that contains all the Feature's assets.  For example, it may be expanded to `/tmp/vsch/container-features/featureB_1`.

Given this example, `featureB`'s file structure would look something like:

```
.
├── src
│   ├── featureB
│   │   ├── bundledScript.sh
│   │   ├── devcontainer-feature.json
│   │   └── install.sh
```

### Installation

Given the following `devcontainer.json` and the two example Features above.

```jsonc
{
    "image": "ubuntu",
    "features": {
        "featureA":  {},
        "featureB":  {},
    },
    "postCreateCommand":  "userPostCreate.sh",
    "postAttachCommand": {
        "server": "npm start",
        "db": ["mysql", "-u", "root", "-p", "my database"]
    },
}
```

The following timeline of events would occur. 

> Note that:
>
>1. Each bullet point below is _blocking_. No subsequent lifecycle commands shall proceed until the current command completes.
> 2.  If one of the lifecycle scripts fails, any subsequent scripts will not be executed. For instance, if a given `postCreate` command fails, the `postStart` hook and any following hooks will be skipped.
>

- `featureA`'s onCreateCommand
- `featureA`'s postCreateCommand 
- `featureB`'s postCreateCommand
-  The user's postCreateCommand
- `featureA`'s postAttachCommand **(parallel syntax, each command running in parallel)**
-  The user's postAttachCommand **(parallel syntax, each command running in parallel)**

Suppose `featureB`'s postCreateCommand exited were to exit unsuccessfully (non-zero exit code).  In this case, the `postAttachCommand` will never fire.
