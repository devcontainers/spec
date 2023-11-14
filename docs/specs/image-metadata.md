# Image Metadata

## Goal

Record dev container config and feature metadata in prebuilt images, such that, the image and the built-in features can be used with a devcontainer.json (image-, Dockerfile- or Docker Compose-based) that does not repeat the dev container config or feature metadata. Other tools should be able to record the same metadata without necessarily using features themselves.

Current dev container config that can be recorded in the image: `mounts`, `onCreateCommand`, `updateContentCommand`, `postCreateCommand`, `postStartCommand`, `postAttachCommand`, `customizations`, `remoteUser`, `userEnvProbe`, `remoteEnv`, `containerEnv`, `overrideCommand`, `portsAttributes`, `otherPortsAttributes`, `forwardPorts`, `shutdownAction`, `updateRemoteUserUID` and `hostRequirements`.

Current feature metadata relevant for using the feature when it is already part of the image: `mounts`, `init`, `privileged`, `capAdd`, `securityOpt`, `entrypoint` and `customizations`.

We can add to these lists as we add more properties to the dev container configuration and the feature metadata.

## Proposal

We propose to add metadata to the image with the following structure using one entry per feature and devcontainer.json (see table below for the full list):

```
[
	{
		"id"?: string,
		"init"?: boolean,
		"privileged"?: boolean,
		"capAdd"?: string[],
		"securityOpt"?: string[],
		"entrypoint"?: string,
		"mounts"?: [],
		...
		"customizations"?: {
			...
		}
	},
	...
]
```

To simplify adding this metadata for other tools, we also support having a single top-level object with the same properties.

The metadata is added to the image as a `devcontainer.metadata` label with a JSON string value representing the above array or single object.

## Merge Logic

To apply the metadata together with a user's devcontainer.json at runtime the following merge logic by property is used. The table also notes which properties are currently supported coming from the devcontainer.json and which from the feature metadata, this will change over time as we add more properties.

| Property | Type/Format | Merge Logic | devcontainer.json | Feature Metadata |
| -------- | ----------- | ----------- | :---------------: | :--------------: |
| `id` | E.g., `ghcr.io/devcontainers/features/node:1` | Not merged. |   | x |
| `init` | `boolean` | `true` if at least one is `true`, `false` otherwise. | x | x |
| `privileged` | `boolean` | `true` if at least one is `true`, `false` otherwise. | x | x |
| `capAdd` | `string[]` | Union of all `capAdd` arrays without duplicates. | x | x |
| `securityOpt` | `string[]` | Union of all `securityOpt` arrays without duplicates. | x | x |
| `entrypoint` | `string` | Collected list of all entrypoints. |   | x |
| `mounts` | `(string \| { type, src, dst })[]` | Collected list of all mountpoints. Conflicts: Last source wins. | x | x |
| `onCreateCommand` | `string \| string[]` | Collected list of all onCreateCommands. | x |   |
| `updateContentCommand` | `string \| string[]` | Collected list of all updateContentCommands. | x |   |
| `postCreateCommand` | `string \| string[]` | Collected list of all postCreateCommands. | x |   |
| `postStartCommand` | `string \| string[]` | Collected list of all postStartCommands. | x |   |
| `postAttachCommand` | `string \| string[]` | Collected list of all postAttachCommands. | x |   |
| `waitFor` | enum | Last value wins. | x |   |
| `customizations` | Object of tool-specific customizations. | Merging is left to the tools. | x | x |
| `containerUser` | `string` | Last value wins. | x |   |
| `remoteUser` | `string` | Last value wins. | x |   |
| `userEnvProbe` | `string` (enum) | Last value wins. | x |   |
| `remoteEnv` | Object of strings. | Per variable, last value wins. | x |   |
| `containerEnv` | Object of strings. | Per variable, last value wins. | x |   |
| `overrideCommand` | `boolean` | Last value wins. | x |   |
| `portsAttributes` | Map of ports to attributes. | Per port (not per port attribute), last value wins. | x |   |
| `otherPortsAttributes` | Port attributes. | Last value wins (not per port attribute). | x |   |
| `forwardPorts` | `(number \| string)[]` | Union of all ports without duplicates. Last one wins (when mapping changes). | x |   |
| `shutdownAction` | `string` (enum) | Last value wins. | x |   |
| `updateRemoteUserUID` | `boolean` | Last value wins. | x |   |
| `hostRequirements` | `cpus`, `memory`, `storage` | Max value wins. | x |   |

Variables in string values will be substituted at the time the value is applied. When the order matters, the devcontainer.json is considered last.

## Additional devcontainer.json Properties

We are adding support for `mounts`, `containerEnv`, `containerUser`, `init`, `privileged`, `capAdd`, and `securityOpt` to the devcontainer.json (also with Docker Compose) the same way these properties are already supported in the feature metadata.

## Notes

- Passing the label as a `LABEL` instruction in the Dockerfile:
	- The size limit on Dockerfiles is around 1.3MB. The line length is limited to 65k characters.
	- Using one line per feature should allow for making full use of these limits.
- Passing the label as a command line argument:
	- There is no size limit documented for labels, but the daemon returns an error when the request header is >500kb.
	- The 500kb limit is shared, so we cannot use a second label in the same build to avoid it.
	- If/when this becomes an issue we could embed the metadata as a file in the image (e.g., with a label indicating it).
