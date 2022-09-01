# Client post-attach commands

`postAttachCommand` is currently a single command which implementations run as part of the container lifecycle.

This proposal makes two changes:
1. Redefines post-attach commands to be a client concern, separate from container lifecycle.
2. Allows multiple, named commands in `object` form. This is to both support multiple post-attach commands and to support using the name as an identifier for client-specific customization.

## Motivation

Teaching & collaboration scenarios want to be able to drop people into devcontainers with the necessary tools & services running in client terminals so that users can see the running commands, interact with them, kill them, restart them, etc.

For example, a project may want to open terminals with its server running in one, a file watcher in another, and a database console in the last:

```json
{
  "postAttachCommand": {
    "server": "npm start",
    "watch": "npm run watch",
    "db": "mysql -u root -p"
  }
}
```

## Spec changes

- `postAttachCommand` will support an `object` form where the key is the unique name of the command and the value is the `string` or `array` command. Duplicate names are unsupport.
- Implementations will no longer run `postAttachCommand` as part of the container lifecycle scripts but instead defer that responsibility to attaching clients. Clients may run the scripts at any point after they are attached and may be run concurrently with container lifecycle scripts.
- Implementations will provide a `read-post-attach-commands` command which will parse the post-attach commands from the config, interpolate variables, and output the final commands. Clients will use this to determine the commands they should run after attaching.

### Example 1: Single unnamed command

**.devcontainer.json**
```jsonc
{
  "postAttachCommand": "echo hello '${containerWorkspaceFolderBasename}'"
}
```

**Command**
```
$ devcontainer read-post-attach-commands --workspace-folder my-project
"echo hello my-project"
```

### Example 2: Multiple named commands

**.devcontainer.json**
```json
{
  "postAttachCommand": {
    "server": ["npm", "start"],
    "hello": "echo hello '${containerWorkspaceFolderBasename}'"
  }
}
```

**Command**
```
$ devcontainer read-post-attach-commands --workspace-folder my-project
{"server":["npm","start"],"hello":"echo hello my-project"}
```
