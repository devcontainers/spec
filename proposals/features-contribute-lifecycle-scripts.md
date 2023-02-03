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

All other semantic match the existing [Lifecycle Scripts](https://containers.dev/implementors/json_reference/#lifecycle-scripts) behavior exactly.

###  Execution

Any Features that declare one of the aforementioned lifecycle hook properties will have their command executed _in parallel with any other Features, or user-contributed, lifecycle scripts_ during the target lifecycle point.

> A consequence of executing scripts in parallel requires that operations in one script  do not block another. The execution order of individual lifecycle scripts within a given lifecycle hook is undefined, and any perceived ordering should not be relied upon.

See here for more information on [lifecycle script parallel execution](https://containers.dev/implementors/spec/#parallel-exec).

## Examples


### Executing a script in the workspace folder

The follow example illustrates contributing an `onCreateCommand` and `postCreateCommand` script to `featureA`.  At each lifecycle hook during the build, the provided script will be executed in the [project workspace folder](https://containers.dev/implementors/spec/#project-workspace-folder), following the same semantics as [user-defined lifecycle hooks](https://containers.dev/implementors/json_reference/#lifecycle-scripts).

```jsonc
{
   "id": "featureA",
   "version": "1.0.0",
   "onCreateCommand": "myOnCreate.sh && myOnCreate2.sh",
   "postCreateCommand": "myPostCreate.sh"
}

```

### Executing a script bundled with the Feature

The following example illustrates executing a `postCreateCommand` script bundled with the Feature utilizing the `${featureRoot}` variable.

```jsonc
{
   "id": "featureB",
   "version": "1.0.0",
   "postCreateCommand": "${featureRoot}/bundledScript.sh"
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
>1. Each bullet point below is _blocking_. No subsequent lifecycle hooks shall proceed until the current hook completes.
> 2.  If one of the lifecycle scripts fails, any subsequent scripts will not be executed. For instance, if `postCreateCommand` fails, `postStartCommand` and any following hooks will be skipped.
>

- `featureA`'s onCreateCommand
- `featureA`'s postCreateCommand, `featureB`'s postCreateCommand, and the user's postCreateCommand **(all executed in parallel)**
-  The user's postCreateCommand **(each command running in parallel)**

Suppose `featureB`'s postCreateCommand exited with a non-zero exit code.  In this case, the `postAttachCommand` will never fire.
