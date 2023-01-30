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

Additionally, introduce a `${featureRoot}` environment variable, which is expanded at build time to the root of the expanded Feature the variable is used from.  

Feature lifecycle hooks will be prepended to the lifecycle hooks (if any) declared by the current `devcontainer.json`. Lifecycle hooks will be prepended in the order that the Feature's installation script was executed.  Commands will be joined in such a way that a failure in any one lifecycle hook, will indicate a failure for that entire lifecycle hook execution.

### Parallel Execution

Any Features that declare lifecycle hooks using the [parallel execution syntax](https://containers.dev/implementors/spec/#parallel-exec) will be executed in parallel with any other Features or user-contributed lifecycle scripts. Any Features that do not use this syntax will continue to be run in sequence, blocking subsequent runs until the script exits successfully.

Each parallel stage inherited from a Feature will be prefixed with the Feature `id`. Eg: `featureA_migrateDB`.

## Examples


### Executing a script in the workspace folder

The follow example illustrates executing a `postCreateCommand` script in the [project workspace folder](https://containers.dev/implementors/spec/#project-workspace-folder).  

```jsonc
{
   "id": "featureA",
   "version": "1.0.0"
   "postCreateCommand": "scriptInWorkspaceFolder.sh"
}

```

### Executing a script bundled with the Feature

The follow example illustrates executing a `postCreateCommand` script bundled with the Feature by using the `${featureRoot}` variable.

```jsonc
{
   "id": "featureB",
   "version": "1.0.0"
   "postCreateCommand": "${featureRoot}/bundledScript.sh",
   "installsAfter": [
        "featureA"
   ]
}
```

At build time, the `${featureRoot}` variable will be expanded to the temporary directory that contains all the Feature's assets.  For example, it may be expanded to `/tmp/vsch/container-features/featureB_1`.

When authored, `featureB`'s file structure would look something like:

```
.
├── src
│   ├── featureB
│   │   ├── bundledScript.sh
│   │   ├── devcontainer-feature.json
│   │   └── install.sh
```

### Installation Order (without parallel execution syntax)

Given the following `devcontainer.json` and the two examples Features above.

```jsonc
{
    "image": "ubuntu",
    "features": {
        "featureA":  {},
        "featureB":  {},
    },
    "postCreateCommand":  "scriptInWorkspaceFolder.sh"
}
```

The three `postCreate` lifecycle events declared or inherited for this dev container would be serially executed (blocking the next script) in the following order:

- featureA
- featureB
- the user's dev container

Note that the current working directory during the execution of all three scripts is equal to the  [project workspace folder](https://containers.dev/implementors/spec/#project-workspace-folder).

### Installation Order (parallel execution)

Given a Feature defined as follows:

```jsonc
{
    "id": "featureC",
    "version": "1.0.0",
    "postCreateCommand": {
        "server": "npm start",
        "db": ["mysql", "-u", "root", "-p", "my database"]
    },
    "installsAfter": [
        "featureA",
        "featureB"
    ]
}
```

The following dev container configuration:

```jsonc
{
    "image": "ubuntu",
    "features": {
        "featureA":  {},
        "featureB":  {},
        "featureC":  {},
    },
    "postCreateCommand": {
        "git": "git clone https://github.com/devcontainers/cli",
        "getTool": "wget https://example.com/tool/"
    }
}
```

With each bullet point indicating a blocking operation for the following scripts, the execution for the `postCreate` lifecycle hook would behave like:

- featureA
- featureB
- In parallel the execution of 
    - `featureC_server`
    - `featureC_db` 
    - `git`
    - `getTool`