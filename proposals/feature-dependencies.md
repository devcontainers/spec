# # Feature Dependencies

Reference: https://github.com/devcontainers/spec/pull/208/files

An alternate and conflicting proposal can be found here: https://github.com/devcontainers/spec/pull/208/.  If one proposal is accepted, the other should be rejected.

## Motivation

We've seen significant interest in the ability to "reuse" or "extend" a given Feature with one or more additional Features.  Often in software a given tool depends on another (or several) tool(s)/framework(s).  As the dev container Feature ecosystem has grown, there has been a growing need to reduce redundant code in Features by first installing a set of dependent Features.

## Goal

The solution shall provide a way to publish a Feature that "depends" on >= 1 other published Features. Dependent Features will be installed by the orchestrating tool following this specification.

The solution outlined shall not only execute the installation scripts, but also merge the additional development container config, as outlined in the documented [merging logic.](https://containers.dev/implementors/spec/#merge-logic).

The solution should provide guidelines for Feature authorship, and prefer solutions that enable simple authoring of Features, as well as easy consumption by end-users.

A non-goal is to require the use or implementation of a full-blown dependency management system (such as `npm` or `apt`).  The solution should allow Features to operate as "self-contained, shareable units of installation code and development container configuration"[(1)](https://containers.dev/implementors/features/). 

## Existing Solutions

See https://github.com/devcontainers/spec/pull/208/files#diff-a29ffaac693437b6fbf001066b97896f7aef4d6d37dc65b8b98b22a5e19f6c7aR26 for existing solutions and alternative approaches.

## Proposed Specification

### (A) Add `dependsOn` property to Feature metadata.

A new property `dependsOn` can be optionally added to the `devcontainer-feature.json`.  This property mirrors the `features` object in `devcontainer.json`.  Adding Feature(s) to this property tell the orchestrating tool to install the Feature(s) before installing the Feature that declares the dependency. 

The installation order is subject to the algorithm set forth in this document. Where there is ambiguity, it is up to the orchestrating tool to decide the order of installation.

| Property | Type | Description |
|----------|------|-------------|
| `dependsOn` | `string` | The ID of the Feature.  This follows the same semantics of the `id` property in the `devcontainer-feature.json` file. |

An example `devcontainer-feature.json` file with a dependency on three other published Features:

```json
{
    "name": "My Feature",
    "id": "myFeature",
    "version": "1.0.0",
    "dependsOn": {
        "ghcr.io/myotherFeature:1": {
            "flag": true
        },
        "features.azurecr.io/aThirdFeature:1": {},
        "features.azurecr.io/aFourthFeature:1.2.3": {}

    }
}
```
