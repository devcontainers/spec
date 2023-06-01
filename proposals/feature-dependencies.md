# Feature Dependencies

Reference: https://github.com/devcontainers/spec/issues/109

**NOTE**: An alternate and conflicting proposal can be found here: https://github.com/devcontainers/spec/pull/208/.  If the aforementioned proposal is accepted, this proposal should be withdrawn (and vice versa).

**NOTE**: An experimental, prototype implementation can be found here: https://github.com/devcontainers/cli/pull/511

## Motivation

We've seen significant interest in the ability to "reuse" or "extend" a given Feature with one or more additional Features.  Often in software a given tool depends on another (or several) tool(s)/framework(s).  As the dev container Feature ecosystem has grown, there has been a growing need to reduce redundant code in Features by first installing a set of dependent Features.

## Goal

The solution shall provide a way to publish a Feature that "depends" on >= 1 other published Features. Dependent Features will be installed by the orchestrating tool following this specification.

The solution outlined shall not only execute the installation scripts, but also merge the additional development container config, as outlined in the documented [merging logic.](https://containers.dev/implementors/spec/#merge-logic).

The solution should provide guidelines for Feature authorship, and prefer solutions that enable simple authoring of Features, as well as easy consumption by end-users.

A non-goal is to require the use or implementation of a full-blown dependency management system (such as `npm` or `apt`).  The solution should allow Features to operate as "self-contained, shareable units of installation code and development container configuration"[(1)](https://containers.dev/implementors/features/). 

## Definitions

- **User-defined Feature**.  Features defined explicitly by the user in the `features` object of `devcontainer.json`.
- **Published Feature**.  Features published to an OCI registry or a direct HTTPS link to a Feature tgz.

## Existing Solutions

See https://github.com/devcontainers/spec/pull/208/files#diff-a29ffaac693437b6fbf001066b97896f7aef4d6d37dc65b8b98b22a5e19f6c7aR26 for existing solutions and alternative approaches.

## Proposed Specification

### (A) Add `dependsOn` property

A new property `dependsOn` can be optionally added to the `devcontainer-feature.json`.  This property mirrors the `features` object in `devcontainer.json`.  Adding Feature(s) to this property tell the orchestrating tool to install the Feature(s) (with the associated options, if provided) before installing the Feature that declares the dependency.

> This property is similar to the existing `installsAfter` property, with the key distinctions that `installsAfter` (1) is **not** recursive, (2) indicates a soft dependency to influence installation order **if and only if a given Feature is already set to be installed via a user-defined Feature or transitively through a user-defined Feature**, and (3) Features indicated by `installsAfter` can not provide options, nor are they able to be pinned to a specific version tag or digest.

The installation order is subject to the algorithm set forth in this document. Where there is ambiguity, it is up to the orchestrating tool to decide the order of installation. Implementing tools should provide a consistent installation order in instances of ambiguity.

| Property | Type | Description |
|----------|------|-------------|
| `dependsOn` | `object` | The ID and options of the Feature dependency.  Pinning to version tags or digests are honored.  When published, the ID must point to either a Feature (1) published to an OCI registry, or (2) a Feature Tgz URI(**).  Otherwise, IDs follow the same semantics of the `features` object in `devcontainer.json`.  Incompatible with the `installsAfter` property. |


(**) Deprecated Feature identifiers (i.e GitHub Release) are not supported and should be considered a fatal error. For [local Features (ie: during development)](https://containers.dev/implementors/features-distribution/#addendum-locally-referenced), you may also depend on other local Features by providing a relative path to the Feature, relative to folder containing the active `devcontainer.json`. This behavior mirrors the `features` object in `devcontainer.json`.

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

`myfeature` will be installed after `myotherFeature`, `aThirdFeature`, and `aFourthFeature`.

The following three sections will illustrate how an orchestrating tool shouuld identify dependencies between Features, depending on [how the Feature is referenced](https://containers.dev/implementors/features/#referencing-a-feature).

#### Identifying Dependencies - OCI Registry

To speed up dependency resolution, all Features [published to an OCI registry](https://containers.dev/implementors/features-distribution/#oci-registry) will have their entire `devcontainer-feature.json` serialized and added as an annotation..  

The orchestrating tool can use this annotation to resolve the Feature's dependencies without having to download and extract the Feature's tarball.

More specifically, an [annotation](https://github.com/opencontainers/image-spec/blob/main/annotations.md) named `dev.containers.metadata` will be populated on the manifest when published by an implementing tool.  This annotation is the escaped JSON object of the entire `devcontainer-feature.json` as it appears during the [packaging stage](https://containers.dev/implementors/features-distribution/#packaging).  

An example manifest with the `dev.containers.metadata` annotation:

```json
{
  "schemaVersion": 2,
  "mediaType": "application/vnd.oci.image.manifest.v1+json",
  "config": {
    "mediaType": "application/vnd.devcontainers",
    "digest": "sha256:e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855",
    "size": 0
  },
  "layers": [
    {
      "mediaType": "application/vnd.devcontainers.layer.v1+tar",
      "digest": "sha256:738af5504b253dc6de51d2cb1556cdb7ce70ab18b2f32b0c2f12650ed6d2e4bc",
      "size": 3584,
      "annotations": {
        "org.opencontainers.image.title": "devcontainer-feature-myFeature.tgz"
      }
    }
  ],
  "annotations": {
    "dev.containers.metadata": "{\"name\": \"My Feature\",\"id\": \"myFeature\",\"version\": \"1.0.0\",\"dependsOn\": {\"ghcr.io/myotherFeature:1\": {\"flag\": true},\"features.azurecr.io/aThirdFeature:1\": {},\"features.azurecr.io/aFourthFeature:1.2.3\": {}}}"
  }
}
```

> If no annotation is present on a Feature's manifest, the orchestrating tool may deduce the Feature has been published prior to this specification and has no declared dependencies.

Supporting tools may choose to identify the dependencies of the declared user-defined Features by fetching the manifest of the Features and reading the `dependsOn` and `installsAfter` properties from the `dev.containers.metadata` annotation.  This will allow the orchestrating tool to recursively resolve all dependencies without having to download and extract each Feature's tarball. 

#### Identifying Dependencies - HTTPS Direct Tarball

Published Features referenced directly by [HTTPS URI pointing to the packaged tarball archive](https://containers.dev/implementors/features-distribution/#directly-reference-tarball) will need to be fully downloaded an extracted to read the `dependsOn` property.

#### Identifying Dependencies - Local Features

[Local Features](https://containers.dev/implementors/features-distribution/#addendum-locally-referenced) dependencies are identified by directly reading the `dependsOn` property in the associated `devcontainer-feature.json` metadata file.

### (B) `dependsOn` install order algorithm

The orchestrating tool is responsible for calculating a Featuure installation order (or error, if no valid installation order can be resolved). The set of Features to be installed is the union of user-defined Features and their dependencies.  The orchestrating tool will perform the following steps:

#### (B1) Building dependency graph

From the user-defined Features, the orchestrating tool will build a dependency graph.  The graph will be built by traversing the `dependsOn` property of each Feature and recursively resolving each Feature from the property.  An accumulator is maintained with each new Feature that has been discovered, as well as a pointer to its dependencies.  If the exact Feature (see **Feature Equality**) has already been added to the accumulator, it will not be added again.  The accumulator will be fed into (B2) after all the Feature tree has been resolved.


#### (B2) Round-based sorting

Perform a sort on the result of (B1) in rounds. This sort will rearrange and deduplicate Features, producing a sorted list of Features to install.  The sort will be performed as follows: 

Start with all the elements from (B1) in a `worklist` and an empty list `installationOrder`.  While the `worklist` is not empty, iterate through each element in the `worklist` and check if all its dependencies (if any) are already members of `installationOrder`.  If the check is true, add it to an intermediate list `round` and remove it from the `worklist`.  If not, skip it.  Equality is determined in **Feature Equality**.

Before committing each `round` to the `installationOrder`, sort all elements of round according to **Round Stable Sort***.

Repeat for as many rounds as necessary until the worklist is empty.  If there is ever a round where no elements are added to `installationOrder`, the algorithm should terminate and return an error.  This indicates a circular dependency or other error in the dependency graph.


#### Definition: Feature Equality

This specification defines two Features as equal if both Features point to the same exact contents and are executed with the same options.

**For Features published to an OCI registry**, two Feature are identical if their manifest digests are equal, and the options executed against the Feature are equal (compared value by value).  Identical manifest digests implies that the tgz contents of the Feature and its entire `devcontainer-feature.json` are identical.  If any of these conditions are not met, the Features are considered not equal.

**For Features fetched by HTTPS URI**, two Features are identical if the contents of the tgz are identical (hash to the same value), and the options executed against the Feature are equal (compared value by value).  If any of these conditions are not met, the Features are considered not equal.

**For local Features**, each Feature is considered unique and not equal to any other local Feature.

#### Definition: Round Stable Sort

To prevent non-deterministic behavior, the algorithm will sort each **round** according to the following rules:

- Compare and sort each Feature lexiographically by their fully qualified resource name (For OCI-published Features, that means the ID without version or digest.).  If the comparison is equal:
- Compare and sort each Feature from oldest to newest tag (`latest` being the "most new").  If the comparision is equal:
- Compare and sort each Feature by their options by:
  - Greatest number of user-defined options (note omitting an option will default that value to the Feature's default value and is not considered a user-defined option). If the comparison is equal:
  - Sort the provided option keys lexicographically.  If the comparison is equal:
  - Sort the provided option values lexicographically. If the comparision is equal:
- Sort Features by their canonical name (For OCI-published Features, the Feature ID resolved to the digest hash).

If there is no difference based on these comparator rules, the Features are considered equal.

## Note: Existing methods of altering Feature installation order

Two existing properties (1) `installsAfter` on the Feature metadata, and  (2) `overrideFeatureInstallationOrder` in the `devcontainer.json` both exist to alter the installation order of user-defined Features.  The behavior of these properties are carried forward in this proposal.

### installsAfter

The `installsAfter` property is a "soft dependency" that influences the installation order of Features that are queued to be installed.  The effective behavior of this property is the same as `dependsOn`, with the following differences:

- `installsAfter` is not recursive.  
- `installsAfter` only influences the installation order of Features that are already set to be installed after resolving the dependency tree.
- The Feature indicated by `installsAfter` can not provide options, nor are they able to be pinned to a specific version tag or digest.  This is unchanged from the current specification.

From an implementation point of view, `installsAfter` nodes may be added as a seperate set of directed edges, just as `dependsOn` nodes are added as directed edges (see **(B1)**).  Before round-based installation and sorting **(B2)**, an orchestrating tool should remove all `installsAfter` directed edges that do not correspond with a Feature in the `worklist` that is set to be installed.  In each round, a Feature can then be installed if all its requirements (both `dependsOn` and `installsAfter` dependencies) have been fulfilled in previous rounds.

An implemention should fail the dependency resolution step if the evaluation of the `installsAfter` property results in an inconsistent state (eg: a circular dependency).

### overrideFeatureInstallOrder

The `overrideFeatureInstallOrder` property of `devcontainer.json` is an array of Feature IDs that are to be installed in descending priority order as soon as its dependencies outlined above are installed.  

This evaluation is performed after **Round Stable Sort** and may result in a different installation order within a round.

This property must not influence the dependency relationship as defined by the dependency graph (see **(B1)**) and shall only be evaulated at the round-based sorting step (see **(B2)**).  Put another way, this property cannot "pull forward" a Feature until all of its dependencies (both soft and hard) have been installed.  After a Feature's dependencies have been installed in other rounds, this property should "pull forward" each Feature as early as possible (given the order of identifiers in the array).

Similar to `installsAfter`, this property's members may not provide options, nor are they able to be pinned to a specific version tag or digest.  This is unchanged from the current specification.

If a Feature is indicated in `overrideFeatureInstallOrder` but not a member of the dependency graph (it is not queued to be installed), the orchestrating tool may fail the dependency resolution step.

One implementation strategy involves assigning each node in the `worklist` a roundPriority integer, defaulting to 0.  If a Feature is indicated in `overrideFeatureInstallOrder`, the orchestrating tool should set the roundPriority of that Feature to  `length(overrideFeatureInstallOrder) - idx`, where `idx` is the zero-based index of the property in the array.  The orchestrating tool should then sort the `worklist` by roundPriority before committing each round's installation order, placing higher priority Features earlier in the `installationOrder`.

## Additional Remarks

### Feature authorship

Features should be authored with the following considerations:

- Features should be authored with the assumption that they will be installed in any order, so long as the dependencies are met at some point beforehand.
- Since two Features with different options are considered different, a single Feature may be installed more than once.  Features should be idempotent.
- Features that require updating shared state in the container (eg: updating the `$PATH`), should be aware that the same Feature may be run multiple times. Consider a method for previous runs of the Feature to communicate with future runs, updating the shared state in the intended way.


### Image Metadata

Dependency resolution is only performed on initial container creation by reading the `features` object of `devcontainer.json` and resolving the Features in that point in time.  For subsequent creations from an image (or resumes of a dev container), the dependency tree is **not** re-calculated.  To re-calculate the dependency tree, the user must delete the image (or dev container) and recreate it from a `devcontainer.json`.

Once Feature dependencies are resolved, they are treated the same as if members of the user-defined Features.  That means both user-defined Features and their dependencies are stored in the image's metadata.