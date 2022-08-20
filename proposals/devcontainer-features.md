# Dev Container Features reference

Development container "Features" are self-contained, shareable units of installation code and development container configuration. The name comes from the idea that referencing one of them allows you to quickly and easily add more tooling, runtime, or library "Features" into your development container for you or your collaborators to use.

> **Note:** While Features may be installed on top of any base image, the implementation of a Feature might restrict it to a subset of possible base images.  
> 
> For example, some Features may be authored to work with a certain linux distro (e.g. debian-based images that use the `apt` package manager).

Feature metadata is captured by a `devcontainer-feature.json` file in the root folder of the feature.

> **Tip:** This section covers details on the Features specification. If you are looking for summarized information on creating your own Features, see the [template](https://github.com/devcontainers/feature-template) and [core Features](https://github.com/devcontainers/features) repositories.

## Folder structure 

A feature is a self contained entity in a folder with at least a `devcontainer-feature.json` and `install.sh` entrypoint script.  Additional files are permitted and are packaged along side the required files.

```
+-- feature
|    +-- devcontainer-feature.json
|    +-- install.sh
|    +-- (other files)
```

## devcontainer-feature.json properties

the `devcontainer-feature.json` file defines information about the feature to be used by any supporting tools and the way the feature will be executed.

The properties of the file are as follows:

| Property | Type | Description |
| :--- | :--- | :--- |
| `id` | string | ID of the feature/definition. The `id` should be unique in the context of the repository/published package where the feature exists and must match the name of the directory where the `devcontainer-feature.json` resides. |
| `version` | string | The semantic version of the feature. |
| `name` | string | Name of the feature/definition. |
| `description` | string | Description of the feature/definition. |
| `documentationURL` | string | Url that points to the documentation of the feature. |
| `licenseURL` | string | Url that points to the license of the feature. |
| `keywords` | array | List of strings relevant to a user that would search for this definition/feature. |
| `options` | object | A map of options that will be passed as environment variables to the execution of the script. |
| `containerEnv` | object | A set of name value pairs that sets or overrides environment variables. |
| `privileged` | boolean | Sets [privileged mode](https://docs.docker.com/engine/reference/run/#runtime-privilege-and-linux-capabilities) for the container (required by things like docker-in-docker) when the feature is used. |
| `init` | boolean | Adds the [tiny init](https://github.com/RKrahl/tiny-init) process to the container (`--init`) when the feature is used. |
| `capAdd` | array | Adds container [capabilities](https://docs.docker.com/engine/security/#linux-kernel-capabilities) when the Feature is used. |
| `securityOpt` | array | Sets container security options like updating the [seccomp profile](https://docs.docker.com/engine/security/seccomp/) when the feature is used. |
| `entrypoint` | string | Set if the feature requires an "entrypoint" script that should fire at container start up. |
| `customizations` | object | Product specific properties, each namespace under `customizations` is treated as a separate set of properties. For each of this sets the object is parsed, values are replaced while arrays are set as a union. |
| `installsAfter` | array | Array of ID's of Features that should execute before this one. Allows control for feature authors on soft dependencies between different Features. |

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

## devcontainer.json properties

Features are referenced in a user's [`devcontainer.json`](/docs/specs/devcontainer-reference.md#devcontainerjson) under the top level `features` object. 

A user can specify an arbitrary number of Features.  At build time, these Features will be installed in an order defined by a combination of the [installation order rules and implementation](#Installation-Order). 

A single feature is provided as a key/value pair, where the key is the feature identifier, and the value is an object containing "options" (or empty for "default").  Each key in the feature object must be unique.

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

> Note: The `:latest` version annotation is added implicitly if omitted. To pin to a specific package version ([example](https://github.com/devcontainers/features/pkgs/container/features/go/versions)), append it to the end of the feature.

An option's value can be provided as either a `string` or `boolean`, and should match what is expected by the feature in the `devcontainer-feature.json` file.

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

The `id` format specified dictates how a supporting tool will locate and download a given feature. `id` is one of the following:

| Type | Description | Example |
| :--- | :--- | :--- |
| `<oci-registry>/<namespace>/<feature>[:<semantic-version>]` | Reference to feature in OCI registry(*) | `ghcr.io/user/repo/go` <br> `ghcr.io/user/repo/go:1` <br> `ghcr.io/user/repo/go:latest`|
| `https://<uri-to-feature-tgz>` | Direct HTTPS URI to a tarball. | `https://github.com/user/repo/releases/devcontainer-feature-go.tgz` |
| `./<path-to-feature-dir>`| A relative directory(**) to folder containing a devcontainer-feature.json. | `./myGoFeature` |

(*) OCI registry must implement the [OCI Artifact Distribution Specification](https://github.com/opencontainers/distribution-spec).  Some implementors can be [found here](https://oras.land/implementors/).

(**) The provided path is always relative to the `.devcontainer/` folder, and is outlined further in the [Locally Referenced Addendum](/proposals/devcontainer-features-distribution.md#Addendum:-Locally-Referenced).

## Versioning

Each feature is individually [versioned according to the semver specification](https://semver.org/).  The `version` property in the respective `devcontainer-feature.json` file is updated to increment the feature's version.

Tooling that handles releasing Features will not republish Features if that exact version has already been published; however, tooling must republish major and minor versions in accordance with the semver specification.

## Authoring

Features can be authored in a number of languages, the most straightforward being bash scripts. If a feature is authored in a different language information about it should be included in the metadata so that users can make an informed choice about it.

Reference information about the application required to execute the feature should be included in `devcontainer-feature.json` in the metadata section.

Applications should default to `/bin/sh` for Features that do not include this information.

If the Feature is included in a folder as part of the repository that contains `devcontainer.json`, no other steps are necessary.

## Release

_For information on distribution features, see [devcontainer-features-distribution.md](./devcontainer-features-distribution.md)._

## Execution

### Installation order

By default, Features are installed on top of a base image in an order determined as optimal by the implementing tool.

If any of the following properties are provided in the feature's `devcontainer-feature.json`, or the user's `devcontainer.json`, the order indicated by these propert(ies) are respected (with decreasing precedence).

1. The `overrideFeatureInstallOrder` property in user's `devcontainer.json`. Allows users to control the order of execution of their Features.
2. The `installsAfter` property defined as part of a feature's `devcontainer-feature.json`.

#### (1) The `overrideFeatureInstallOrder` devcontainer.json property

This property is declared by the user in their `devcontainer.json` file.

Any un-versioned feature IDs listed in this array will be installed before all other Features, in the provided order. Any omitted Features will be installed in an order selected by the implementing tool, or ordered via the `installsAfter` property _after_  any Features listed in the `overrideFeatureInstallOrder` array, if applicable. 

All un-versioned Feature `id`s provided in `overrideFeatureInstallOrder` must also exist in the `features` property of a user's `devcontainer.json`. For instance, all the Features which follows the OCI registry format would include everything except for the label that contains the version (`<oci-registry>/<namespace>/<feature>` without the `:<semantic-version>`.

Example:
```
  "features": {
      "ghcr.io/devcontainers/features/java:1",
      "ghcr.io/devcontainers/features/node:1",
  },
  "overrideFeatureInstallOrder": [
    "ghcr.io/devcontainers/features/node"
  ]
```

| Property | Type | Description |
| :--- | :--- | :--- |
| `overrideFeatureInstallOrder` | array | Array consisting of the feature `id` (without the semantic version) of Features in the order the user wants them to be installed.   |

#### (2) The `installsAfter` Feature property

This property is defined in an individual feature's `devcontainer-feature.json` file by the feature author.  `installsAfter` allows an author to provide the tooling hints on loose dependencies between Features.

After `overrideFeatureInstallOrder` is resolved, any remaining Features that declare an `installsAfter` must be installed after the Features declared in the property, provided that the Features have also been declared in the `features` property.

| Property | Type | Description |
| :--- | :--- | :--- |
| `installsAfter` | array | Array consisting of the feature `id` that should be installed before the given Feature |

### Option Resolution

A feature's 'options' - specified as the value of a single feature key/value pair in the user's `devcontainer.json` - are passed to the feature as environment variables.

A supporting tool will parse the `options` object provided by the user.  If a value is provided for a feature, it will be emitted to a file named `devcontainer-features.env` following the format `<OPTION_NAME>=<value>`.  

To ensure a option that is valid as an environment variable, the follow substitutions are performed.

```javascript
(str: string) => str
	.replace(/[^\w_]/g, '_')
	.replace(/^[\d_]+/g, '_')
	.toUpperCase();
```

This file is sourced at build-time for the feature `install.sh` entrypoint script to handle.

Any options defined by a feature's `devcontainer-feature.json` that are omitted in the user's `devcontainer.json` will be implicitly exported as its default value.

### Option resolution example

Suppose a `python` feature has the following `options` parameters declared in the `devcontainer-feature.json` file:

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

The user's `devcontainer.json` declared the python feature like so

```jsonc

"features": {
    "python": {
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

...outputs the following:
```
Version is 3.10
Pip? false
Optimize? true
```

### Implementation notes

There are several things to keep in mind for an application that implements Features:

- The order of execution of Features is determined by the application, based on the `installAfter` property used by feature authors. It can be overridden by users if necessary with the `overrideFeatureInstallOrder` in `devcontainer.json`.
- Features are used to create an image that can be used to create a container or not.
- Parameters like `privileged`, `init` are included if just 1 feature requires them.
- Parameters like `capAdd`, `securityOp`  are concatenated.
- `containerEnv` is added before the feature is executed as `ENV` commands in the Dockerfile.
- Each feature script executes as its own layer to aid in caching and rebuilding.
- There is also a proposal to move properties like `privileged` to `devcontainer.json` for consistency and to reduce reliance on the `runArgs` property. See https://github.com/devcontainers/spec/issues/2
