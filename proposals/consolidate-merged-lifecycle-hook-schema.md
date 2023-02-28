# Goal

Expand the lifecycle hook `devcontainer.json` schema to make it possible to represent a merged configuration within a `devcontainer.json`.   This will allow a configuration comprised of lifecycle hooks comprised of [Feature contributed lifeycycle scripts](./features-contribute-lifecycle-scripts.md) or otherwise merged onto the image to not need a separate 'mergedConfiguration' schema to represent the config.

## Motivation

As seen below, the current generated 'mergedConfiguration' returned by the `read-configuration` CLI command does not return a `devcontainer.json` as outlined in this repo's specification, but rather an array of hooks. [(source)](https://github.com/devcontainers/cli/pull/390#issuecomment-1430190326)

![img-1](https://user-images.githubusercontent.com/23246594/218825633-cf037d97-db05-4d0d-9157-66287cd47073.png)

## Proposal

Update the schema of lifecycle hooks to mirror the merged configuration illustrated above.

Propose to change the interface to:

```typescript

export type LifecycleCommand = string | string[] | { [key: string]: string | string[] };
// export type LifecycleCommandEntry = LifecycleCommand | {lifeCycleCommand: LifecycleCommand; origin: string}

interface DevContainerConfig {
    // ...other devcontainer.json properties...
	onCreateCommand?: LifecycleCommand | LifecycleCommand[];
	updateContentCommand?: LifecycleCommand | LifecycleCommand[];
	postCreateCommand?: LifecycleCommand | LifecycleCommand[];
	postStartCommand?: LifecycleCommand | LifecycleCommand[];
	postAttachCommand?: LifecycleCommand | LifecycleCommand[];
}
```