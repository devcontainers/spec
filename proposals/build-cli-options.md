# [Proposal] Allow additional cli options to pass to the build command

Related to:
- https://github.com/devcontainers/spec/discussions/301
- https://github.com/devcontainers/spec/pull/324
- https://github.com/microsoft/vscode-remote-release/issues/3545

## Goal

Extend the build section of the configuration with an option to pass additional arguments via cli.

## Proposal

Introduce the property `cliOptions` to the `build` property in the schema as type `Array (string)`. This field contains elements that are directly passed as cli-arguments to the `build` command. It is similar to `runArgs` property which contains cli-arguments passed to the `run` command.

###  Execution

When a dev container is built, all entries from the `cliOptions` are passed to the `build` command after all other options.

## Examples

### Add a host

The following example adds a host-definition while the image is being built:

```jsonc
{
  "build": {
    "dockerfile": "Dockerfile",
    "cliOptions": [
      "--add-host=host.docker.internal:host-gateway"
    ]
  },
}
```

### Use host network

The following example allows to sue the host network while building an image:

```jsonc
{
  "build": {
    "dockerfile": "Dockerfile",
    "cliOptions": [
      "--network=host"
    ]
  },
}
```
