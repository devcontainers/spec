# GPU host requirement

## Goal

This proposal adds support for GPU to the existing support for host requirements.
- Issue: https://github.com/devcontainers/spec/issues/82
- Community PR: https://github.com/devcontainers/cli/pull/173

## Proposal

We propose to add a new `gpu` property to the `hostRequirements` object in the devcontainer.json schema. The property can be a boolean value, the string `optional` or an object:

(Additional row for the existing table in the spec.)
| Property | Type | Description |
|----------|------|-------------|
| `hostRequirements.gpu` 🏷️ | boolean \| 'optional' | Indicates whether or not a GPU is required. A value 'optional' indicates that might be used if available. The object is described in the table below. The default is false. For example: `"hostRequirements": "optional"` |

The object value can have the following properties:

| Property | Type | Description |
|----------|------|-------------|
| `hostRequirements.gpu.cores` 🏷️ | integer | Indicates the minimum required number of cores. For example: `"hostRequirements": { "gpu": {"cores": 2} }` |
| `hostRequirements.gpu.memory` 🏷️ | string |  A string indicating minimum memory requirements with a `tb`, `gb`, `mb`, or `kb` suffix. For example, `"hostRequirements": {"memory": "4gb"}` |
