# Goal


This proposal aims to raise the 'mergedConfiguration' into the specification, so that implementing tools can generate `devcontainer.json`, as well as consume, a spec-compliant 'mergedConfiguration' by parsing a user 'devcontainer.json'.  Properties that can not or should not be represented as `devcontainer.json` properties are prepended with a `$`.  This proposal standardizes how these properies should be outputted, so that supporting tooos can consume them consistently.

## Motivation

The current generated 'mergedConfiguration' returned by the `read-configuration` CLI command does not return a `devcontainer.json` as outlined in this repo's specification. There are several inconsistencies. [(source)](https://github.com/devcontainers/cli/pull/390#issuecomment-1430190326)

By expanding the lifecycle hook `devcontainer.json` schema as outlined below, it is possible to represent a merged configuration within a `devcontainer.json`.  This, for example, will allow a configuration comprised of [Feature contributed lifecycle scripts](./features-contribute-lifecycle-scripts.md) to be represented as a `devcontainer.json` without needing to add non-spec properties (ie: plural `postCreateCommands`).

## Proposal

### A. Update the devcontainer.json schema

#### A1. Change the Lifecycle Script Interface

Add a value (`LifecycleCommandParallel[]`) to the unioned definition of each lifecycle hook (except 'initializeCommand').

> NOTE: _ This was chosen as every existing lifecycle command (and merged version of each command) can be represented within that type._

```typescript

export type LifecycleCommand = string | string[]
export type LifecycleCommandParallel = { [key: string]: LifecycleCommand; $origin?: string };

interface DevContainerConfig {
    // ...other devcontainer.json properties...
	onCreateCommand?: LifecycleCommand | LifecycleCommandParallel | LifecycleCommandParallel[]
	updateContentCommand?: LifecycleCommand | LifecycleCommandParallel | LifecycleCommandParallel[]
	postCreateCommand?: LifecycleCommand | LifecycleCommandParallel | LifecycleCommandParallel[]
	postStartCommand?: LifecycleCommand | LifecycleCommandParallel | LifecycleCommandParallel[]
	postAttachCommand?: LifecycleCommand | LifecycleCommandParallel | LifecycleCommandParallel[]
}
```

An optional parameter `$origin` is added to the `LifecycleCommandParallel` that supporting tools can use to indicate the source of the command.  For example, this is useful for outputting in the creation log which Feature provided a certain lifecycle hook.  The `$` notation is used to indicate this property is additional tooling metadata that should not be present in a user `devcontainer.json`.

As an example, the following `devcontainer.json` snippet would be valid:

```json
{
	"onCreateCommand": [
		{
			"commandA": "echo 'Hello World!'",
			"commandB": ["echo", "'Hello World!'"],
			"$origin": "featureA"
		},
		{
			"commandA": "echo 'Hello World!'",
			"commandB": ["echo", "'Hello World!'"],
			"$origin": "devcontainer.json"
		}
	]
}
```

### B. 'read-configuration' command to generate a merged configuration.

The [reference implementation](https://github.com/devcontainers/cli) includes a 'read-configuration' command that can be used to resolve a fully merged configuration from a given `devcontainer.json`.

Implementing tools should follow the [documented merging logic](https://containers.dev/implementors/spec/#merge-logic) and output a `devcontainer.json` that represents the resolved configuration.

There are a couple special cases to the merging logic that should be noted:

#### B1. The `customizations` property

The merging of the `customizations` property is left to the implementing tool, therefore there is no defined merging pattern. 

A property `$customizations` should be added to the merged configuration as an array of customizations within each top-level customization object.  Tooling that reads the mergedConfiguration can process this property as needed.

In the below example, the `vscode` customization has two entries (contributed from two different sources), and the `foo` customization has two entries (contributed from two different sources).  The merged `$customizations` property would be an array of two objects, one for each customization namespace.

```json
"$customizations": {
	// Customizations for the 'vscode' namespace
	"vscode": [
		{
			"settings": {
			"settingA": "local",
			"settingB": "/usr/bin/lldb"
			},
			"extensions": [
			"GitHub.vscode-pull-request-github"
			]
		},
		{
			"settings": {
			"settingA": "local",
			"settingC": true
			}
		}
	],
	// Customizations for the 'foo' namespace
	"foo": [
		{
			"bar": "baz",
			"b": true
		},
		{
			"bar": "baz",
			"a": true
		}
	]
},
...
```

### B2. The `entrypoint` property

Dev Container Features are able to contribute an `entrypoint` property. This property is not available in the `devcontainer.json`.

A property `$entrypoints` should be added to the merged configuration containing an array of entrypoint strings.  Tooling that reads the mergedConfiguration can process this property as needed.

In the below example, the `entrypoint` property is contributed from two different sources.  The merged `$entrypoints` property would be an array of two strings, one for each entrypoint.

```json
"$entrypoints": [
	"/usr/bin/entrypointA",
	"/usr/bin/entrypointB"
],
```

## Summary

With these additions, the 'mergedConfiguration' returned by `read-configuration` can directly return a `devcontainer.json` without needing non-spec properties (ie: plural `postCreateCommands`).  The addition of `$origin` in the `LifecycleCommandParallel` type allows for tooling to indicate the source of a lifecycle hook.  The addition of `$customizations` and `$entrypoints` allows for tooling to indicate the source of a customization or entrypoint without needing to add non-schema properties to the `devcontainer.json`.

