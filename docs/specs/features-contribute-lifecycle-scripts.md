# Allow Dev Container Features to contribute lifecycle scripts

This has now been implemented:
* Discussion issue: https://github.com/devcontainers/spec/issues/60, https://github.com/devcontainers/spec/issues/181
* CLI PR: https://github.com/devcontainers/cli/pull/390

Below is the original proposal.

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

As with all lifecycle hooks, commands are executed from the context (cwd) of the [project workspace folder](https://containers.dev/implementors/spec/#project-workspace-folder). 

> **Note:** To use any assets (a script, etc...) embedded within a Feature in a lifecycle hook property, it is the Feature author's reponsibility to copy that assert to somewhere on the container where it will persist (outside of `/tmp`.) Implementations are not required to persist Feature install scripts beyond the initial build.

All other semantics match the existing [Lifecycle Scripts](https://containers.dev/implementors/json_reference/#lifecycle-scripts) and [lifecycle script parallel execution](https://containers.dev/implementors/spec/#parallel-exec) behavior exactly.

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

The following example illustrates executing a `postCreateCommand` script bundled with the Feature.

```jsonc
{
   "id": "featureB",
   "version": "1.0.0",
   "postCreateCommand": "/usr/myDir/bundledScript.sh",
   "installsAfter": [
        "featureA"
   ]
}
```

The `install.sh` will need to move this script to `/usr/myDir`.

```
#!/bin/sh

cp ./bundleScript /usr/myDir
...
...
```

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
