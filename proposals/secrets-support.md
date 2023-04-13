# First class secrets support in devcontainers CLI

## What are secrets
Secrets are variables that hold sensitive values and needs to be handled securely at all times (API keys, passwords etc.). Users can change a secret's value at anytime, and the devcontainers CLI should support dynamically changing secrets without having to rebuild the container.

## Goal

Support Secrets as a first class feature in devcontainers CLI.

This feature should comprise of the ability to securely pass in the secrets in CLI commands, make them available for users to consume similar to remoteEnv and containerEnv, and be processed and handled securely at all times.
This functionality will allow consumers to be able to use secrets more predictably and securely with devcontainers CLI.

## Motivation

Today Codespaces pass secret variables as `remoteEnv` since there are no other mechanisms to pass them explicitly as secrets. The devcontainers CLI do not (and need not) treat `remoteEnv` as secrets. Having explicit support for secrets will help eliminating security threats such as leaking secrets, as well as in improving customer confidence.

## Proposal

A [supporting tool](https://containers.dev/supporting#tools) (ie, the [dev container CLI reference implementation](https://github.com/devcontainers/cli) should behave according to the following properties:

  1. Ability to pass secrets to commands
  2. Apply/use secrets similar to `remoteEnv`
  3. Securely handle secrets

### Passing secrets to commands
Secrets are not part of devcontainers specification and we do not expect users to store secrets inside devcontainer.json file.

Secrets should be passed to the following devcontainers CLI commands.

#### **Phase 1**
 - up
 - run-user-commands

`up`, `run-user-commands` will use the secrets for usecases like lifecycle hooks (similar to remoteEnv). We do not plan secrets support for `build` command since we do not have a demostrated requirement, as well as to avoid potentially baking in the user secrets into the docker images.

#### **Phase 2**
In phase 2 the support for secrets can be expanded to additional commands such as `set-up` and `exec`.

#### **Input format**
Best way to pass secrets would be through a json/env file, where in the file path is supplied in a command line argument `--secrets-file`.

Today the devcontaners CLI [commands accept](https://github.com/devcontainers/cli/blob/5c81479f0342947dd3e52a5984b9150e5feb8fd6/src/spec-node/devContainersSpecCLI.ts#L114) remote environment variables of the format name=value. However that format may not work as it is here, since in many cases the secret values may span multiple lines. One way to workaround that problem is by base64 encoding the values.

Example secrets file:
```
API_KEY=YWRzamhzZDZkZndkamZ3ZGU3ZWR3ZndlZGZkamVkd2Y3d2VkZndl
NUGET_CONFIG=PGNvbmZpZz4KICAgIDxhZGQga2V5PSJkZXBlbmRlbmN5VmVyc2lvbiIgdmFsdWU9IkhpZ2hlc3QiIC8+CiAgICA8YWRkIGtleT0iaHR0cF9wcm94eSIgdmFsdWU9Imh0dHA6Ly9jb21wYW55LXNxdWlkOjMxMjhAY29udG9zby5jb20iIC8+CjwvY29uZmlnPg==
PASSWORD=U2ltcGxlIFBhc3N3b3Jkcw==
```

Example secrets file in json format:
```json
[
  	{
		"name": "API_KEY",
		"value": "adsjhsd6dfwdjfwde7edwfwedfdjedwf7wedfwe"
	},
	{
		"name": "NUGET_CONFIG",
		"value": "<config>\n    <add key=\"dependencyVersion\" value=\"Highest\" />\n    <add key=\"http_proxy\" value=\"http://company-squid:3128@contoso.com\" />\n</config>"
	},
	{
		"name": "PASSWORD",
		"value": "Simple Passwords"
	}
]
```

Further simplified json kv pair format (Preffered):
```json
{
	"API_KEY": "adsjhsd6dfwdjfwde7edwfwedfdjedwf7wedfwe",
	"NUGET_CONFIG": "<config>\n    <add key=\"dependencyVersion\" value=\"Highest\" />\n    <add key=\"http_proxy\" value=\"http://company-squid:3128@contoso.com\" />\n</config>",
	"PASSWORD": "Simple Passwords"
}
```

### Using secrets in the CLI
Today Codespaces as a consumer is leveraging `remoteEnv` to pass secrets into the CLI. So it is a basic requirement that secrets should be used and behave the same way as `remoteEnv` do.

### Securely handle secrets
Secrets should be handled securely at all times.
- Secrets should not be passed as plain text in command line
- Do not be persist secrets on the container labels or long lived permanent caches
- Redact secrets to prevent them being written in plain text into logs and console output.
