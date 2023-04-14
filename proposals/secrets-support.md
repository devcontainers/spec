# First class secrets support in dev containers implementations

## What are secrets
Secrets are variables that hold sensitive values and need to be handled securely at all times (API keys, passwords etc.). Users can change a secret's value at any time, and a conforming dev containers implementation should support dynamically changing secrets without having to rebuild the container.

## Goal

Support secrets as a first class feature in dev containers implementations.

This feature should consist of the ability to securely pass in the secrets, make them available for users to consume similar to remoteEnv and containerEnv, and be processed and handled securely at all times.
This functionality will allow consumers to be able to use secrets more predictably and securely with their dev containers implementation.

## Motivation

Today many consumers pass secret variables as `remoteEnv`, since there are no other mechanisms to pass them explicitly as secrets. The dev containers reference implementation do not (and need not) treat `remoteEnv` as secrets. Having explicit support for secrets will help eliminate security threats such as leaking secrets, as well as in improving customer confidence.

## Proposal

A [supporting tool](https://containers.dev/supporting#tools) (ie, for example the [dev container CLI reference implementation](https://github.com/devcontainers/cli)) should behave according to the following properties:

  1. Ability to pass secrets to commands
  2. Apply/use secrets similar to `remoteEnv`
  3. Securely handle secrets

### Passing secrets in
Secrets are not part of dev containers specification and we do not expect users to store secrets inside devcontainer.json file. It is recommended to pass the secrets in separate a file.

In the future the support could be expanded to accept secrets from additional sources such as Windows credential manager, Mac keychain, Azure keyvault etc.

#### **Input format**

Using `env` files can be a basic and simple option. However that format may not work as it is in some cases, for example the secret values may often span multiple lines. One way to workaround that problem is by base64 encoding the values.

1. Example secrets env file:
	```
	API_KEY=YWRzamhzZDZkZndkamZ3ZGU3ZWR3ZndlZGZkamVkd2Y3d2VkZndl
	NUGET_CONFIG=PGNvbmZpZz4KICAgIDxhZGQga2V5PSJkZXBlbmRlbmN5VmVyc2lvbiIgdmFsdWU9IkhpZ2hlc3QiIC8+CiAgICA8YWRkIGtleT0iaHR0cF9wcm94eSIgdmFsdWU9Imh0dHA6Ly9jb21wYW55LXNxdWlkOjMxMjhAY29udG9zby5jb20iIC8+CjwvY29uZmlnPg==
	PASSWORD=U2ltcGxlIFBhc3N3b3Jkcw==
	```

2. Example secrets file in json format:
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

3. Further simplified json kv pair format:
	```json
	{
		"API_KEY": "adsjhsd6dfwdjfwde7edwfwedfdjedwf7wedfwe",
		"NUGET_CONFIG": "<config>\n    <add key=\"dependencyVersion\" value=\"Highest\" />\n    <add key=\"http_proxy\" value=\"http://company-squid:3128@contoso.com\" />\n</config>",
		"PASSWORD": "Simple Passwords"
	}
	```

### Using secrets
Today many consumers are leveraging `remoteEnv` to pass in secrets. So it is a basic requirement that secrets should be used and behave the same way as `remoteEnv` do.

### Securely handle secrets
Secrets should be handled securely at all times.
- Secrets should not be exposed in plan text at rest and in transit.
- Do not persist secrets on the container labels or long lived permanent caches
- Redact/mask secrets before writing to logs and output.

## Notes
Notes on how this proposal may be adopted by the dev containers CLI (reference implementation).

### Passing secrets to the commands
Secrets should not be passed directly in the command line since that would risk it getting stored in command history. Best way to pass secrets would be through a json/env file, where in the file path is supplied in a command line argument named `--secrets-file`.

Secrets should be passed to the following dev containers CLI commands.

#### **Phase 1**
 - up
 - run-user-commands

`up`, `run-user-commands` will use the secrets for use cases like lifecycle hooks (similar to remoteEnv). We do not plan secrets support for `build` command since we do not have a demonstrated requirement, as well as to avoid potentially baking in the user secrets into the docker images.

#### **Phase 2**
In phase 2 the support for secrets can be expanded to additional commands such as `set-up` and `exec`.
