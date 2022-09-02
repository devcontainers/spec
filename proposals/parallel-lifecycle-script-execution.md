# Parallel lifecycle script execution

Support executing multiple lifecycle scripts in parallel by providing them in `object` form.

## Motivation

Devcontainers supports a single command for each of its lifecycle scripts. While serial execution of multiple commands can be achieved with `;`, `&&`, etc. parallel is less straightforward and so deserves first-class support.

## Spec changes

All lifecycle scripts will be extended to support `object` types. The key of the `object` will be a unique name for the command and the value will be the `string` or `array` command. Each command must exit successfully for the stage to be considered successful.

### Example

```json
{
  "postCreateCommand": {
    "server": "npm start",
    "db": ["mysql", "-u", "root", "-p", "my database"]
  }
}
```
