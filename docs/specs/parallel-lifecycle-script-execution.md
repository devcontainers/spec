# Parallel Lifecycle Script Execution

This has now been implemented:
* Discussion issue: https://github.com/devcontainers/spec/issues/83
* CLI PR: https://github.com/devcontainers/cli/pull/172

Below is the original proposal.

## Goal

Support executing multiple lifecycle scripts in parallel by providing them in `object` form.

## Motivation

Dev containers support a single command for each of their lifecycle scripts. While serial execution of multiple commands can be achieved with `;`, `&&`, etc. parallel is less straightforward and so deserves first-class support.

## Spec changes

All lifecycle scripts will be extended to support `object` types. The key of the `object` will be a unique name for the command and the value will be the `string` or `array` command. Each command must exit successfully for the stage to be considered successful.

Each entry in the `object` will be run in parallel during that lifecycle step.

### Example

```json
{
  "postCreateCommand": {
    "server": "npm start",
    "db": ["mysql", "-u", "root", "-p", "my database"]
  }
}
```
