# First Class Secrets Support

This has now been implemented:
* Discussion issue: https://github.com/devcontainers/spec/issues/219
* CLI PR: https://github.com/devcontainers/cli/pull/493

Below is the original proposal.

## What are secrets
Secrets are variables that hold sensitive values and need to be handled securely at all times (API keys, passwords etc.). Users can change a secret's value at any time, and a conforming dev containers implementation should support dynamically changing secrets without having to rebuild the container.

## Goal

Support secrets as a first class feature in dev containers implementations.

This feature should consist of the ability to securely pass in the secrets, make them available for users to consume similar to remoteEnv and containerEnv, and be processed and handled securely at all times.
This functionality will allow consumers to be able to use secrets more predictably and securely with their dev containers implementation.

## Motivation

Today many consumers pass secret variables as `remoteEnv`, since there are no other mechanisms to pass them explicitly as secrets. The dev containers reference implementation do not (and need not) treat `remoteEnv` as secrets. Having explicit support for secrets will help eliminate security threats such as leaking secrets, as well as in improving customer confidence.

## Proposal

A [supporting tool](https://containers.dev/supporting#tools) (e.g. the [dev container CLI reference implementation](https://github.com/devcontainers/cli)) should behave according to the following properties:

  1. Ability to pass secrets to commands
  2. Apply/use secrets similar to `remoteEnv`
  3. Securely handle secrets

### Passing secrets in
Secrets are not part of dev containers specification and we do not expect users to store secrets inside `devcontainer.json` file. A conforming implementation should provide a secure mechanism to pass secrets, such as a secrets file, Windows credential manager, Mac keychain, Azure keyvault etc. for example.

#### **Example**

Using a file to pass in the secrets can be one of the simple approaches to adopt. In this example the supporting tool can input secrets in a JSON file format.

```json
{
	"API_KEY": "adsjhsd6dfwdjfwde7edwfwedfdjedwf7wedfwe",
	"NUGET_CONFIG": "<config>\n    <add key=\"dependencyVersion\" value=\"Highest\" />\n    <add key=\"http_proxy\" value=\"http://company-squid:3128@contoso.com\" />\n</config>",
	"PASSWORD": "Simple Passwords"
}
```
