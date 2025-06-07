# Dev Container Features reference

**Development Container Features** are self-contained, shareable units of installation code and development container configuration. The name comes from the idea that referencing one of them allows you to quickly and easily add more tooling, runtime, or library "Features" into your development container for you or your collaborators to use.

> **Note:** While Features may be installed on top of any base image, the implementation of a Feature might restrict it to a subset of possible base images.  
> 
> For example, some Features may be authored to work with a certain linux distro (e.g. debian-based images that use the `apt` package manager).

Feature metadata is captured by a `devcontainer-feature.json` file in the root folder of the Feature.

> **Tip:** This section covers details on the Features specification. If you are looking for summarized information on creating your own Features, see the [template](https://github.com/devcontainers/feature-template) and [core Features](https://github.com/devcontainers/features) repositories.

## Folder structure 

A Feature is a self contained entity in a folder with at least a `devcontainer-feature.json` and `install.sh` entrypoint script. Additional files are permitted and are packaged along side the required files.

```
+-- feature
|    +-- devcontainer-feature.json
|    +-- install.sh
|    +-- (other files)
```

## `devcontainer-feature.json` properties

The `devcontainer-feature.json` file defines metadata about a given Feature.

All properties are optional **except for `id`, `version`, and `name`**. 

| Property | Type | Description |
| :--- | :--- | :--- |
| `id` | string | **Required**: Identifier of the Feature.  Must be unique in the context of the repository where the Feature exists and must match the name of the directory where the `devcontainer-feature.json` resides. ID should be provided lowercase. |
| `version` | string | **Required**: The semantic version of the Feature (e.g: `1.0.0`). |
| `name` | string | **Required**: A "human-friendly" display name for the Feature. |
| `description` | string | Description of the Feature. |
| `documentationURL` | string | Url that points to the documentation of the Feature. |
| `licenseURL` | string | Url that points to the license of the Feature. |
| `keywords` | array | List of strings relevant to a user that would search for this Feature. |
| `options` | object | A map of options that will be passed as environment variables to the execution of the script. |
| `containerEnv` | object | A set of name value pairs that sets or overrides environment variables. |
| `privileged` | boolean | Sets [privileged mode](https://docs.docker.com/engine/reference/run/#runtime-privilege-and-linux-capabilities) for the container (required by things like docker-in-docker) when the Feature is used. |
| `init` | boolean | Adds the [tini init](https://github.com/krallin/tini) process to the container (`--init`) when the Feature is used. |
| `capAdd` | array | Adds container [capabilities](https://docs.docker.com/engine/security/#linux-kernel-capabilities) when the Feature is used. |
| `securityOpt` | array | Sets container security options like updating the [seccomp profile](https://docs.docker.com/engine/security/seccomp/) when the Feature is used. |
| `entrypoint` | string | Set if the Feature requires an "entrypoint" script that should fire at container start up. |
| `customizations` | object | Product specific properties, each namespace under `customizations` is treated as a separate set of properties. For each of this sets the object is parsed, values are replaced while arrays are set as a union. |
| `dependsOn` | `object` | An object (\**)  of Feature dependencies that **must** be satisified before this Feature is installed. Elements follow the same semantics of the `features` object in `devcontainer.json`. [See *Installation Order* for further information](#installation-order).  |
| `installsAfter` | array | Array of ID's of Features (omitting a version tag) that should execute before this one. Allows control for Feature authors on soft dependencies between different Features. [See *Installation Order* for further information](#installation-order). |
| `legacyIds` | array | Array of old IDs used to publish this Feature. The property is useful for renaming a currently published Feature within a single namespace. |
| `deprecated` | boolean | Indicates that the Feature is deprecated, and will not receive any further updates/support. This property is intended to be used by the supporting tools for highlighting Feature deprecation. |
| `mounts` | object | Defaults to unset. Cross-orchestrator way to add additional mounts to a container. Each value is an object that accepts the same values as the [Docker CLI `--mount` flag](https://docs.docker.com/engine/reference/commandline/run/#mount). The Pre-defined [devcontainerId](./devcontainerjson-reference.md/#variables-in-devcontainerjson) variable may be referenced in the value. For example:<br />`"mounts": [{ "source": "dind-var-lib-docker", "target": "/var/lib/docker", "type": "volume" }]` |

(**) The ID must refer to either a Feature (1) published to an OCI registry, (2) a Feature Tgz URI, or (3) a Feature in the local file tree. Deprecated Feature identifiers (i.e GitHub Release) are not supported and the presence of this property may be considered a fatal error or ignored. For [local Features (ie: during development)](https://containers.dev/implementors/features-distribution/#addendum-locally-referenced), you may also depend on other local Features by providing a relative path to the Feature, relative to folder containing the active `devcontainer.json`. This behavior of Features within this property again mirror the `features` object in `devcontainer.json`.

### Lifecycle Hooks

The following lifecycle hooks may be declared as properties of `devcontainer-feature.json`. 

| Property | Type|
| :--- | :--- |
| `onCreateCommand` | [string, array, object](/docs/specs/devcontainerjson-reference.md#formatting-string-vs-array-properties)|
| `updateContentCommand` | [string, array, object](/docs/specs/devcontainerjson-reference.md#formatting-string-vs-array-properties)|
| `postCreateCommand` | [string, array, object](/docs/specs/devcontainerjson-reference.md#formatting-string-vs-array-properties)|
| `postStartCommand` | [string, array, object](/docs/specs/devcontainerjson-reference.md#formatting-string-vs-array-properties) |
| `postAttachCommand` | [string, array, object](/docs/specs/devcontainerjson-reference.md#formatting-string-vs-array-properties) |

#### Behavior

Each property mirrors the behavior of the matching property in [`devcontainer.json`](/docs/specs/devcontainerjson-reference.md#Lifecycle-scripts), including the behavior that commands are executed from the context of the [project workspace folder](https://containers.dev/implementors/spec/#project-workspace-folder).

For each lifecycle hook (in [Feature installation order](https://containers.dev/implementors/features/#installation-order)), each command contributed by a Feature is executed in sequence (blocking the next command from executing).   Commands provided by Features are always executed _before_ any user-provided lifecycle commands (i.e: in the `devcontainer.json`).

If a Feature provides a given command with the [object syntax](/docs/specs/devcontainerjson-reference.md#formatting-string-vs-array-properties), all commands within that group are executed in parallel, but still blocking commands from subsequent Features and/or the `devcontainer.json`.

> NOTE: These properties are stored within [image metadata](https://github.com/devcontainers/spec/blob/joshspicer/lifecycle_hook_feature_spec/docs/specs/devcontainer-reference.md#merge-logic).

### The `options` property

The options property is contains a map of option IDs and their related configuration settings. The ID becomes the name of the environment variable in all caps. See [option resolution](#option-resolution) for more details. For example:

```json
{
  "options": {
    "optionIdGoesHere": {
      "type": "string",
      "description": "Description of the option",
      "proposals": ["value1", "value2"],
      "default": "value1"
    }
  }
}
```

| Property | Type | Description |
| :--- | :--- | :--- |
| `optionId` | string | ID of the option that is converted into an all-caps environment variable with the selected value in it. |
| `optionId.type` | string | Type of the option. Valid types are currently: `boolean`, `string` |
| `optionId.proposals` | array | A list of suggested string values. Free-form values **are** allowed. Omit when using `optionId.enum`. |
| `optionId.enum` | array | A strict list of allowed string values. Free-form values are **not** allowed. Omit when using `optionId.proposals`. |
| `optionId.default` | string or boolean | Default value for the option. |
| `optionId.description` | string | Description for the option. |

### User environment variables

Feature scripts run as the `root` user and sometimes need to know which user account the dev container will be used with.

`_REMOTE_USER` and `_CONTAINER_USER` environment variables are passed to the Features scripts with `_CONTAINER_USER` being the container's user and `_REMOTE_USER` being the configured `remoteUser`. If no `remoteUser` is configured, `_REMOTE_USER` is set to the same value as `_CONTAINER_USER`.

Additionally, the home folders of the two users are passed to the Feature scripts as `_REMOTE_USER_HOME` and `_CONTAINER_USER_HOME` environment variables.

The container user can be set with `containerUser` in the devcontainer.json and image metadata, `user` in the docker-compose.yml, `USER` in the Dockerfile, and can be passed down from the base image.

### Dev Container ID

An identifier will be referred to as `${devcontainerId}` in the `devcontainer.json` and the Feature metadata and that will be replaced with the dev container's id. It should only be used in parts of the configuration and metadata that is not used for building the image because that would otherwise prevent pre-building the image at a time when the dev container's id is not known yet. Excluding boolean, numbers and enum properties the properties supporting `${devcontainerId}` in the Feature metadata are: `entrypoint`, `mounts`, `customizations`.

Implementations can choose how to compute this identifier. They must ensure that it is unique among other dev containers on the same Docker host and that it is stable across rebuilds of dev containers. The identifier must only contain alphanumeric characters. We describe a way to do this below.

#### Label-based Implementation 

The following assumes that a dev container can be identified among other dev containers on the same Docker host by a set of labels on the container. Implementations may choose to follow this approach.

The identifier is derived from the set of container labels uniquely identifying the dev container. It is up to the implementation to choose these labels. E.g., if the dev container is based on a local folder the label could be named `devcontainer.local_folder` and have the local folder's path as its value.

E.g., the [`ghcr.io/devcontainers/features/docker-in-docker` Feature](https://github.com/devcontainers/features/blob/main/src/docker-in-docker/devcontainer-feature.json) could use the dev container id with:

```jsonc
{
    "id": "docker-in-docker",
    "version": "1.0.4",
    // ...
    "mounts": [
        {
            "source": "dind-var-lib-docker-${devcontainerId}",
            "target": "/var/lib/docker",
            "type": "volume"
        }
    ]
}
```

#### Label-based Computation

- Input the labels as a JSON object with the object's keys being the label names and the object's values being the labels' values.
	- To ensure implementations get to the same result, the object keys must be sorted and any optional whitespace outside of the keys and values must be removed.
- Compute a SHA-256 hash from the UTF-8 encoded input string.
- Use a base-32 encoded representation left-padded with '0' to 52 characters as the result.

JavaScript implementation taking an object with the labels as argument and returning a string as the result:

```js
const crypto = require('crypto');
function uniqueIdForLabels(idLabels) {
	const stringInput = JSON.stringify(idLabels, Object.keys(idLabels).sort()); // sort properties
	const bufferInput = Buffer.from(stringInput, 'utf-8');
	const hash = crypto.createHash('sha256')
		.update(bufferInput)
		.digest();
	const uniqueId = BigInt(`0x${hash.toString('hex')}`)
		.toString(32)
		.padStart(52, '0');
	return uniqueId;
}
```

## devcontainer.json properties

Features are referenced in a user's [`devcontainer.json`](/docs/specs/devcontainer-reference.md#devcontainerjson) under the top level `features` object. 

A user can specify an arbitrary number of Features. At build time, these Features will be installed in an order defined by a combination of the [installation order rules and implementation](#Installation-Order). 

A single Feature is provided as a key/value pair, where the key is the Feature identifier, and the value is an object containing "options" (or empty for "default"). Each key in the Feature object must be unique.

These options are sourced as environment variables at build-time, as specified in [Option Resolution](#Option-Resolution).

Below is a valid `features` object provided as an example.
```jsonc
"features": {
  "ghcr.io/user/repo/go": {},
  "ghcr.io/user/repo1/go:1": {},
  "ghcr.io/user/repo2/go:latest": {},
  "https://github.com/user/repo/releases/devcontainer-feature-go.tgz": { 
        "optionA": "value" 
  },
  "./myGoFeature": { 
        "optionA": true,
        "optionB": "hello",
        "version" : "1.0.0"
  }
}
```

> Note: The `:latest` version annotation is added implicitly if omitted. To pin to a specific package version ([example](https://github.com/devcontainers/features/pkgs/container/features/go/versions)), append it to the end of the Feature.

An option's value can be provided as either a `string` or `boolean`, and should match what is expected by the Feature in the `devcontainer-feature.json` file.

As a shorthand, the value of a `feature` can be provided as a single string. This string is mapped to an option called `version`.  In the example below, both examples are equivalent. 

```jsonc
"features": {
  "ghcr.io/owner/repo/go": "1.18"
}
```
```jsonc
"features": {
  "ghcr.io/owner/repo/go": {
    "version": "1.18"
  }
}
```

### Referencing a feature 

The `id` format specified dictates how a supporting tool will locate and download a given Feature. `id` is one of the following:

| Type | Description | Example |
| :--- | :--- | :--- |
| `<oci-registry>/<namespace>/<feature>[:<semantic-version>]` | Reference to feature in OCI registry(*) | `ghcr.io/user/repo/go` <br> `ghcr.io/user/repo/go:1` <br> `ghcr.io/user/repo/go:latest`|
| `https://<uri-to-feature-tgz>` | Direct HTTPS URI to a tarball. | `https://github.com/user/repo/releases/devcontainer-feature-go.tgz` |
| `./<path-to-feature-dir>`| A relative directory(**) to folder containing a devcontainer-feature.json. | `./myGoFeature` |

Feature identifiers should be assumed to be case-insensitive and should be normalized to lowercase.

(*) OCI registry must implement the [OCI Artifact Distribution Specification](https://github.com/opencontainers/distribution-spec).  Some implementors can be [found here](https://oras.land/implementors/).

(**) The provided path is always relative to the folder containing the `devcontainer.json`.  Further requirements are outlined in the [Locally Referenced Addendum](./devcontainer-features-distribution.md#locally-referenced-features)

## Versioning

Each Feature is individually [versioned according to the semver specification](https://semver.org/).  The `version` property in the respective `devcontainer-feature.json` file is updated to increment the Feature's version.

Tooling that handles releasing Features will not republish Features if that exact version has already been published; however, tooling must republish major and minor versions in accordance with the semver specification.

## Authoring

Features can be authored in a number of languages, the most straightforward being bash scripts. If a Feature is authored in a different language information about it should be included in the metadata so that users can make an informed choice about it.

Reference information about the application required to execute the Feature should be included in `devcontainer-feature.json` in the metadata section.

Applications should default to `/bin/sh` for Features that do not include this information.

If the Feature is included in a folder as part of the repository that contains `devcontainer.json`, no other steps are necessary.

## Release

_For information on distributing Features, see [devcontainer-features-distribution.md](./devcontainer-features-distribution.md)._

## Execution

### Invoking `install.sh`

The `install.sh` script for each Feature should be executed as `root` during a container image build. This allows the script to add needed OS dependencies or settings that could not otherwise be modified. This also allows the script to switch into another user's context using the `su` command (e.g., `su ${USERNAME} -c "command-goes-here"`). In combination, this allows both root and non-root image modifications to occur even if `sudo` is not present in the base image for security reasons.

To ensure that the appropriate shell is used, the execute bit should be set on `install.sh` and the file invoked directly (e.g. `chmod +x install.sh && ./install.sh`). 

> **Note:** It is recommended that Feature authors write `install.sh` using a shell available by default in their supported distributions (e.g., `bash` in Debian/Ubuntu or Fedora, `sh` in Alpine). In the event a different shell is required (e.g., `fish`), `install.sh` can be used to boostrap by checking for the presence of the desired shell, installing it if needed, and then invoking a secondary script using the shell.
>
> The `install.sh` file can similarly be used to bootstrap something written in a compiled language like Go. Given the increasing likelihood that a Feature needs to work on both x86_64 and arm64-based devices (e.g., Apple Silicon Macs), `install.sh` can detect the current architecture (e.g., using something like `uname -m` or `dpkg --print-architecture`), and then invoke the right executable for that architecture.

### Installation order

By default, Features are installed on top of a base image in an order determined as optimal by the implementing tool.

If any of the following properties are provided in the Feature's `devcontainer-feature.json`, or the user's `devcontainer.json`, the order indicated by these propert(ies) are respected.

* The `dependsOn` property defined as a part of a Feature's `devcontainer-feature.json`.
* The `installsAfter` property defined as part of a Feature's `devcontainer-feature.json`.
* The `overrideFeatureInstallOrder` property in user's `devcontainer.json`. Allows users to control the order of execution of their Features.

#### The `dependsOn` property

The optional `dependsOn` property indicates a set of required, "hard" dependencies for a given Feature.  

The `dependsOn` property is declared in a Feature's `devcontainer-feature.json` metadata file. Elements of this property mirror the semantics of the `features` object in `devcontainer.json`.  Therefore, all dependencies may provide the relevant options, or an empty object (eg: `"bar:123": {}`) if the Feature's default options are sufficient.  Identical Features that provide different options are treated as _different_ Features (see [Feature equality](#definition-feature-equality) for more info).

All Features indicated in the `dependsOn` property **must** be satisfied (a Feature [equal](#definition-feature-equality) to each dependency is present in the installation order) _before_ the given Feature is set to be installed.  If any of the Features indicated in the `dependsOn` property cannot be installed (e.g due to circular dependency, failure to resolve the Feature, etc) the entire dev container creation should fail.

The `dependsOn` property must be evaluated recursively.  Therefore, if a Feature dependency has its own `dependsOn` property, that Feature's dependencies must also be satisfied before the given Feature is installed.

```json
{
    "name": "My Feature",
    "id": "myFeature",
    "version": "1.0.0",
    "dependsOn": {
        "foo:1": {
            "flag": true
        },
        "bar:1.2.3": {},
        "baz@sha256:a4cdc44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855" {},
    }
}
```

In the snippet above, `myfeature` MUST be installed after `foo`, `bar`, and `baz`.  If the Features provided via the `dependsOn` property declare their own dependencies, those must also be satisfied before the Feature is installed.

#### The `installsAfter` Property

The `installsAfter` property indicates a "soft dependency" that influences the installation order of Features that are already queued to be installed. The effective behavior of this property is the same as `dependsOn`, with the following differences:

- `installsAfter` is **not** evaluated recursively.
- `installsAfter` only influences the installation order of Features that are **already set to be installed**.  Any Feature not set to be installed after (1) resolving the `dependsOn` dependency tree or (2) indicated by the user's `devcontainer.json` should not be added to the installation list.
- The Feature indicated by `installsAfter` can **not** provide options, nor are they able to be pinned to a specific version tag or digest.  Resolution to the canonical name should still be performed (eg: If the Feature has been [renamed](#steps-to-rename-a-feature)).

```
{
    "name": "My Feature",
    "id": "myFeature",
    "version": "1.0.0",
    "installsAfter": [
        "foo",
        "bar"
    ]
}
```

In the snippet above, `myfeature` must be installed after `foo` and `bar` **if** the Feature is already queued to be installed. If `second` and `third` are not already queued to be installed, this dependency relationship should be ignored.

#### The 'overrideFeatureInstallOrder' property

The `overrideFeatureInstallOrder` property of `devcontainer.json` is an array of Feature IDs that are to be installed in descending priority order as soon as its dependencies outlined above are installed.

> This property may not indicate an installation order that is inconsistent with the resolved dependency graph (see [dependency algorithm](#dependency-installation-order-algorithm)).  If the `overrideFeatureInstallOrder` property is inconsistent with the dependency graph, the implementing tool should fail the dependency resolution step.

This evaluation is performed by assigning a [`roundPriority`](#2-assigning-round-priority) to all nodes that match match the Feature identifier (version omitted) present in the property. 

For example, given `n` Features in the `overrideFeatureInstallOrder` array, the orchestrating tool should assign a `roundPriority` of `n - idx` to each Feature, where `idx` is the zero-based index of the Feature in the array.

For example:

```javascript
overrideFeatureInstallOrder = [
  "foo",
  "bar",
  "baz"
]
```

would result in the following `roundPriority` assignments:

```javascript
const roundPriority = {
  "foo": 3,
  "bar": 2,
  "baz": 1
}
```

This property must not influence the dependency relationship as defined by the dependency graph (see [dependency graph](#1-build-a-dependency-graph)) and shall only be evaulated at the round-based sorting step (see [round sort](#3-round-based-sorting)).  Put another way, this property cannot "pull forward" a Feature until all of its dependencies (both soft and hard) have been installed. After a Feature's dependencies have been installed in other rounds, this property should "pull forward" each Feature as early as possible (given the order of identifiers in the array).

Similar to `installsAfter`, this property's members may not provide options, nor are they able to be pinned to a specific version tag or digest.

If a Feature is indicated in `overrideFeatureInstallOrder` but not a member of the dependency graph (it is not queued to be installed), the orchestrating tool may fail the dependency resolution step.

> ## Definitions
> ### Definition: Feature Equality
>
> This specification defines two Features as equal if both Features point to the same exact contents and are executed with > the same options.
> 
> **For Features published to an OCI registry**, two Feature are identical if their manifest digests are equal, and the > options executed against the Feature are equal (compared value by value).  Identical manifest digests implies that the tgz  contents of the Feature and its entire `devcontainer-feature.json` are identical.  If any of these conditions are not met,  the Features are considered not equal.
> 
> **For Features fetched by HTTPS URI**, two Features are identical if the contents of the tgz are identical (hash to the > same value), and the options executed against the Feature are equal (compared value by value).  If any of these conditions  are not met, the Features are considered not equal.
> 
> **For local Features**, each Feature is considered unique and not equal to any other local Feature.
> 
> ### Definition: Round Stable Sort
> 
> To prevent non-deterministic behavior, the algorithm will sort each **round** according to the following rules:
> 
> - Compare and sort each Feature lexiographically by their fully qualified resource name (For OCI-published Features, that  means the ID without version or digest.).  If the comparison is equal:
> - Compare and sort each Feature from oldest to newest tag (`latest` being the "most new").  If the comparision is equal:
> - Compare and sort each Feature by their options by:
>   - Greatest number of user-defined options (note omitting an option will default that value to the Feature's default value  and is not considered a user-defined option). If the comparison is equal:
>   - Sort the provided option keys lexicographically.  If the comparison is equal:
>   - Sort the provided option values lexicographically. If the comparision is equal:
> - Sort Features by their canonical name (For OCI-published Features, the Feature ID resolved to the digest hash).
> 
> If there is no difference based on these comparator rules, the Features are considered equal.
>
> 

> ## Dependency installation order algorithm
>
> An implementing tool is responsible for calculating the Feature installation order (or providing an error if no valid installation order can be resolved). The set of Features to be installed is the union of user-defined Features (those directly indicated in the user's `devcontainer.json` and their dependencies (those indicated by the `dependsOn` or `installsAfter` property, taking into account the user dev container's `overrideFeatureInstallOrder` property).  The implmenting tool will perform the following steps:
> ### (1) Build a dependency graph
> 
> From the user-defined Features, the orchestrating tool will build a dependency graph.  The graph will be built by traversing the `dependsOn` and `installsAfter` properties of each Feature.  The metadata for each dependency is then fetched and the node added as an edge to to the dependent Feature.  For `dependsOn`  dependencies, the dependency will be fed back into the worklist to be recursively resolved. 
> 
> An accumulator is maintained with all uniquely discovered and user-provided Features, each with a reference to its dependencies.  If the exact Feature (see **Feature Equality**) has already been added to the accumulator, it will not be added again.  The accumulator will be fed into (B3) after the Feature tree has  been resolved.
> 
> The graph may be stored as an adjacency list with two kinds of edges (1) `dependsOn` edges or "hard dependencies" and (2) `installsAfter` edges or "soft dependencies".
> 
> ### (2) Assigning round priority
> 
> Each node in the graph has an implicit, default `roundPriority` of 0.
> 
> To influence installation order globally while still honoring the dependency graph of built in **(1)**, `roundPriority` values may be tweaks for each Feature.  When each round is calculated in **(3)**, only the Features equal to the max `roundPriority` of that set will be committed (the remaining will be > uncommitted and reevaulated in subsequent rounds).
> 
> The `roundPriority` is set to a non-zero value in the following instances:
> 
> - If the [`devcontainer.json` contains an `overrideFeatureInstallOrder`](#the-overridefeatureinstallorder-property).
> 
> #### (3) Round-based sorting
> 
> Perform a sort on the result of **(1)** in rounds. This sort will rearrange Features, producing a sorted list of Features to install.  The sort will be performed as follows: 
> 
> Start with all the elements from **(2)** in a `worklist` and an empty list `installationOrder`.  While the `worklist` is not empty, iterate through each element in the `worklist` and check if all its dependencies (if any) are already members of `installationOrder`.  If the check is true, add it to an intermediate  list `round` If not, skip it.  Equality is determined in **Feature Equality**.
> 
> Then for each intermediate `round` list, commit to `installationOrder` only those nodes who share the maximum `roundPriority`.  Return all nodes in `round` with a strictly lower `roundPriority` to the `worklist` to be reprocessed in subsequent iterations.  If there are multiple nodes with the same `roundPriority`,  commit them to `installationOrder` with a final sort according to **Round Stable Sort**.
> 
> Repeat for as many rounds as necessary until `worklist` is empty.  If there is ever a round where no elements are added to `installationOrder`, the algorithm should terminate and return an error.  This indicates a circular dependency or other fatal error in the dependency graph.  Implementations should attempt to  provide the user with information about the error and possible mitigation strategies.
>
> ### Notes
>
> From an implementation point of view, `installsAfter` nodes may be added as a separate set of directed edges, just as `dependsOn` nodes are added as directed edges (see **(1)**).  Before round-based installation and sorting **(3)**, an orchestrating tool should remove all `installsAfter` directed edges that do not correspond with a Feature in the `worklist` that is set to be installed.  In each round, a Feature can then be installed if all its requirements (both `dependsOn` and `installsAfter` dependencies) have been fulfilled in previous rounds.
> 
> An implemention should fail the dependency resolution step if the evaluation of the `installsAfter` property results in an inconsistent state (eg: a circular dependency).
>


### Option Resolution

A Feature's 'options' - specified as the value of a single Feature key/value pair in the user's `devcontainer.json` - are passed to the Feature as environment variables.

A supporting tool will parse the `options` object provided by the user.  If a value is provided for a Feature, it will be emitted to a file named `devcontainer-features.env` following the format `<OPTION_NAME>=<value>`.  

To ensure a option that is valid as an environment variable, the follow substitutions are performed.

```javascript
(str: string) => str
	.replace(/[^\w_]/g, '_')
	.replace(/^[\d_]+/g, '_')
	.toUpperCase();
```

This file is sourced at build-time for the Feature `install.sh` entrypoint script to handle.

Any options defined by a Feature's `devcontainer-feature.json` that are omitted in the user's `devcontainer.json` will be implicitly exported as its default value.

### Option resolution example

Suppose a `python` Feature has the following `options` parameters declared in the `devcontainer-feature.json` file:

```jsonc
// ...
"options": {
    "version": {
        "type": "string",
        "enum": ["latest", "3.10", "3.9", "3.8", "3.7", "3.6"],
        "default": "latest",
        "description": "Select a Python version to install."
    },
    "pip": {
        "type": "boolean",
        "default": true,
        "description": "Installs pip"
    },
    "optimize": {
        "type": "boolean",
        "default": true,
        "description": "Optimize python installation"
    }
}
```

The user's `devcontainer.json` declared the python Feature like so

```jsonc

"features": {
    "ghcr.io/devcontainers/features/python:1": {
        "version": "3.10",
        "pip": false
    }
}
```
The emitted environment variables will be:

```env
VERSION="3.10"
PIP="false"
OPTIMIZE="true"
```

These will be sourced and visible to the `install.sh` entrypoint script.  The following `install.sh`...


```bash
#!/usr/bin/env bash

echo "Version is $VERSION"
echo "Pip? $PIP"
echo "Optimize? $OPTIMIZE"
```

... outputs the following:
```
Version is 3.10
Pip? false
Optimize? true
```

### Steps to rename a Feature

1. Update the Feature [source code](./devcontainer-features-distribution.md#source-code) folder and the `id` property in the [devcontainer-feature.json properties](#devcontainer-featurejson-properties) to reflect the new `id`. Other properties (`name`, `documentationUrl`, etc.) can optionally be updated during this step.
2. Add or update the `legacyIds` property to the Feature, including the previously used `id`.
3. Bump the semantic version of the Feature.  
4. Rerun the `devcontainer features publish` command, or equivalent tool that implements the [Features distribution specification](./features-distribution/#distribution).

#### Example: Renaming a Feature

Let's say we currently have a `docker-from-docker` Feature 👇 

Current `devcontainer-feature.json` : 

```jsonc
{
    "id": "docker-from-docker",
    "version": "2.0.1",
    "name": "Docker (Docker-from-Docker)",
    "documentationURL": "https://github.com/devcontainers/features/tree/main/src/docker-from-docker",
    ....
}
```

We'd want to rename this Feature to `docker-outside-of-docker`. The source code folder of the Feature will be updated to `docker-outside-of-docker` and the updated `devcontainer-feature.json` will look like 👇 

```jsonc
{
    "id": "docker-outside-of-docker",
    "version": "2.0.2",
    "name": "Docker (Docker-outside-of-Docker)",
    "documentationURL": "https://github.com/devcontainers/features/tree/main/src/docker-outside-of-docker",
    "legacyIds": [
        "docker-from-docker"
    ]
    ....
}
```

**Note** - The semantic version of the Feature defined by the `version` property should be **continued** and should not be restarted at `1.0.0`.

### Implementation notes

There are several things to keep in mind for an application that implements Features:

- The order of execution of Features is determined by the application, based on the `installsAfter` property used by feature authors. It can be overridden by users if necessary with the `overrideFeatureInstallOrder` in `devcontainer.json`.
- Features are used to create an image that can be used to create a container or not.
- Parameters like `privileged`, `init` are included if just 1 Feature requires them.
- Parameters like `capAdd`, `securityOp`  are concatenated.
- `containerEnv` is added before the Feature is executed as `ENV` commands in the Dockerfile.
- Each Feature script executes as its own layer to aid in caching and rebuilding.
