# Goal

This proposal aims to raise a concept of the 'mergedConfiguration' into the specification, so that implementing tools can generate, as well as consume, a 'merged' `devcontainer.json` file that is the computed merge after parsing a user 'devcontainer.json'.  [The merging logic is already documented](https://containers.dev/implementors/spec/#merge-logic). This proposal focuses on standardizing an output JSON format to represent this merge.

Properties that can not or should not be represented as `devcontainer.json` properties are prepended with a `$`.  This proposal standardizes how these properties should be processed and outputted to file, so that implementing tools can write/consume them consistently.

## Motivation

The current generated 'mergedConfiguration' returned by the `read-configuration` CLI command does not return a `devcontainer.json`. [(visualization)](https://github.com/devcontainers/cli/pull/390#issuecomment-1430190326).  By standardizing the output format, implementing tools can generate a portable, merged `devcontainer.json` that can be consumed by other tools.

## Proposal

### A. Update the devcontainer.json schema

#### A1. Extends lifecycle script properties

Add a value (`LifecycleCommandParallel[]`) to the unioned definition of each lifecycle hook (except 'initializeCommand').  This allows for a syntax that captures the existing allowed types for a lifecycle hook and additionally provides a pattern for attaching additional metadata to a merged configuration.

```typescript
LifecycleCommand = string | string[]
LifecycleCommandParallel = { [key: string]: LifecycleCommand; $origin?: string };
```

In a `devcontainer.json`, each of the following lifecycle hooks shown below can now be represented by one of these three types.

```typescript
{
    // ...other devcontainer.json properties...
	onCreateCommand?: LifecycleCommand | LifecycleCommandParallel | LifecycleCommandParallel[]
	updateContentCommand?: LifecycleCommand | LifecycleCommandParallel | LifecycleCommandParallel[]
	postCreateCommand?: LifecycleCommand | LifecycleCommandParallel | LifecycleCommandParallel[]
	postStartCommand?: LifecycleCommand | LifecycleCommandParallel | LifecycleCommandParallel[]
	postAttachCommand?: LifecycleCommand | LifecycleCommandParallel | LifecycleCommandParallel[]
}
```

An optional parameter `$origin` is added to the `LifecycleCommandParallel` that supporting tools can use to indicate the source of the command.  For example, this is useful for outputting in the creation log which Feature provided a certain lifecycle hook.  The `$` notation is used to indicate this property is additional tooling metadata that should not be present in a user `devcontainer.json`.

> Per [the 'Allow Dev Container Features to contribute lifecycle scripts' specification](/proposals/features-contribute-lifecycle-scripts.md), each `LifecycleCommandParallel` element in the list variant will execute sequentially, in list order.  

As an example, the following `devcontainer.json` snippet would be valid:

```json
{
	"onCreateCommand": [
		{
			"commandA": "echo 'Hello World!'",
			"commandB": ["echo", "'Hello World!'"],
			"$origin": "ghcr.io/devcontainers/featureA@sha256:1234567890"
		},
		{
			"commandA": "echo 'Hello World!'",
			"commandB": ["echo", "'Hello World!'"],
			"$origin": "devcontainer.json"
		}
	]
}
```

#### A2. Extend `customizations` property

The merging of the `customizations` property is left to the implementing tool, therefore there is no defined merging pattern. 

Extend the `devcontainer.json` such that each top-level `customization` namespace is either a map or an array of maps.  The former represents the existing syntax and the latter allows for a namespace's merging algorithm to be defined by the implementing tool.

The addition of the 'array of maps' syntax allows for various tools to contribute customizations to the same namespace.  For example, a Feature may contribute a `vscode` object, and a user (via `devcontainer.json`) may contribute a `vscode` object.  The merged `customizations` property would be an array of two objects.  This allows for the merging algorithm to be defined by the implementing tool.

Similar to above, the `$origin` property may be added by tools to indicate the source of the customization.

In the below example, the `vscode` customization has two entries (contributed from two different sources), and the `foo` customization has two entries (contributed from two different sources).  The merged `customizations` property would be an array of two objects for each customization namespace.

```json
"customizations": {
	// Customizations for the 'vscode' namespace
	"vscode": [
		{
			"settings": {
				"settingA": "local",
				"settingB": "/usr/bin/lldb"
			},
			"extensions": [
				"GitHub.vscode-pull-request-github"
			],
			"$origin": "ghcr.io/devcontainers/featureA@sha256:1234567890"
		},
		{
			"settings": {
				"settingA": "local",
				"settingC": true
			},
			"$origin": "devcontainer.json"
		}
	],
	// Customizations for the 'foo' namespace
	"foo": [
		{
			"bar": "baz",
			"b": true,
			"$origin": "ghcr.io/devcontainers/featureA@sha256:1234567890"
		},
		{
			"bar": "baz",
			"a": true,
			"$origin": "devcontainer.json"
		}
	]
}
...
```

### B. Generating a merged configuration

The [reference implementation](https://github.com/devcontainers/cli) includes a 'read-configuration' command that can be used to resolve a fully merged configuration from a given `devcontainer.json`.

Special cases are detailed below.

### B1. The `entrypoint` property

Dev Container Features are able to contribute an `entrypoint` property. This property is not available in the `devcontainer.json`.

A property `$entrypoints` should be added to the merged configuration containing an array of entrypoint objects.  Each object contains a required `entrypoint` string property and an optional `$origin` meta property.  Tooling that reads the 'mergedConfiguration' can process this property as needed.

In the below example, the `entrypoint` property is contributed from two different sources.  The merged `$entrypoints` property represents the set of active entrypoints.  The optional `$origin` property may be added by tools to indicate the source of the entrypoint.

```json
"$entrypoints": [
	{
		"entrypoint": "/usr/bin/entrypointA",
		"$origin": "ghcr.io/devcontainers/featureA@sha256:1234567890"
	},
	{
		"entrypoint": "/usr/bin/entrypointB",
		"$origin": "ghcr.io/devcontainers/featureB@sha256:1234567890"
	},
],
```

## Summary

With these additions, the 'mergedConfiguration' returned by `read-configuration` can directly return a `devcontainer.json` without needing non-spec properties (ie: plural `postCreateCommands`).  The addition of `$origin` in the `LifecycleCommandParallel` and `customizations` array variant types allows for tooling to indicate the source of provided functionality.  The addition of `$entrypoints` allows for tooling to output metadata on the contributed entrypoint(s) without needing to add non-schema properties to the `devcontainer.json`.

 