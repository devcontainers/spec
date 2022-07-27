# Dev container features reference

Dev container 'features' are self-contained units of installation code and development container configuration. Features are designed to install atop a wide-range of base container images. 
Features are generally designed to be installed on top of a subset of base container images (eg: debian-based images).

Feature metadata is captured by a `devcontainer-feature.json` file in the root folder of the feature.

## Folder Structure 

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
| id | string | Id of the feature/definition. The id should be unique in the context of the repository/published package where the feature exists and must match the name of the directory where the `devcontainer-feature.json` resides. |
| version | string | The semantic version of the feature. |
| name | string | Name of the feature/definition. |
| description | string | Description of the feature/definition. |
| documentationURL | string | Url that points to the documentation of the feature. |
| licenseURL | string | Url that points to the license of the feature. |
| keywords | array | List of keywords relevant to a user that would search for this definition/feature. |
| options | object | Possible options to be passed as environment variables to the execution of the scripts |
| containerEnv | object | A set of name value pairs that sets or overrides environment variables. |
| privileged | boolean | If privileged mode is required by the feature. |
| init | boolean | If it's necessary to run `init`. |
| capAdd | array | Additional capabilities needed by the feature. |
| securityOpt | array | Security options needed by the feature. |
| entrypoint | string | Set if the feature requires an entrypoint. |
| customizations | object | Product specific properties, each namespace under `customizations` is treated as a separate set of properties. For each of this sets the object is parsed, values are replaced while arrays are set as a union. |
| installsAfter | array | Array of Id's of features that should execute before this one. Allows control for feature authors on soft dependencies between different features. |

Options

| Property | Type | Description |
| :--- | :--- | :--- |
| id | string | Id of the option. |
| id.type | string | Type of the option. |
| id.enum | string array | Possible values. |
| id.default | string | Default value for the option. |
| id.description | string | Description for the option. |

## devcontainer.json properties

Features are referenced in a user's [`devcontainer.json`](/docs/specs/devcontainer-reference.md#devcontainerjson) under the top level `features` object. 

A user can specify an arbitrary number of features.  At build time, these features will be installed in an order defined by a combination of the [installation order rules and implementation](#Installation-Order). 

A single feature is provided as a key/value pair, where the key is the feature identifier, and the value is an object containing "options" (or empty for "default").  Each key in the feature object must be unique.

These options are sourced as environment variables at build-time, as specified in [Option Resolution](#Option-Resolution).


Below is a valid `features` object provided as an example.
```jsonc
"features": {
  "ghcr.io/user/repo/go:1": {},
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

The `id` format specified dicates how a supporting tool will locate and download a given feature. `id` is one of the following:

| Type | Description | Example |
| :--- | :--- | :--- |
| `<oci-registry>/<namespace>/<feature>[:<semantic-version>]` | Reference to feature in OCI registry(*) | ghcr.io/user/repo/go:1 |
| `https://<uri-to-feature-tgz>` | Direct HTTPS URI to a tarball. | https://github.com/user/repo/releases/devcontainer-feature-go.tgz |
| `./<path-to-feature-dir>`| A relative directory to folder containing a devcontainer-feature.json. | ./myGoFeature |

`
(*) OCI registry must implement the [OCI Artifact Distribution Specification](https://github.com/opencontainers/distribution-spec).  Some implementors can be [found here](https://oras.land/implementors/).

## Versioning

Each feature is individually [versioned according to the semver specification](https://semver.org/).  The `version` property in the respective `devcontainer-feature.json` file is updated to increment the feature's version.

Tooling that handles releasing features will not republish features if that exact version has already been published; however, tooling must republish major and minor versions in accordance with the semver specification.

## Authoring

Features can be authored in a number of languages, the most straightforward being bash scripts. If a feature is authored in a different language information about it should be included in the metadata so that users can make an informed choice about it.

Reference information about the application required to execute the feature should be included in `devcontainer-feature.json` in the metadata section.

Applications should default to `/bin/sh` for features that do not include this information.

If the feature is included in a folder as part of the repository that contains `devcontainer.json`, no other steps are necessary.

## Release

_For information on distribution features, see [devcontainer-features-distribution.md](./devcontainer-features-distribution.md)._

## Execution

### Installation Order

By default, features are installed on top of a base image in an order determined as optimal by the implementing tool.

If any of the following properties are provided in the feature's `devcontainer-feature.json`, or the user's `devcontainer.json`, the order indicated by these propert(ies) are respected (with decreasing precedence).

1. The `overrideFeatureInstallOrder` property in user's `devcontainer.json`. Allows users to control the order of execution of their features.
2. The `installsAfter` property defined as part of a feature's `devcontainer-feature.json`.

#### (1) overrideFeatureInstallOrder

This property is declared by the user in their `devcontainer.json` file.

Any feature IDs listed in this array will be installed before all other features, in the provided order. Any omitted features will be installed in an order selected by the implementing tool, or ordered via the `installsAfter` property _after_  any features listed in the `overrideFeatureInstallOrder` array, if applicable. 

All feature `id` provided in `overrideFeatureInstallOrder` must also exist in the `features` property of a user's `devcontainer.json`.

| Property | Type | Description |
| :--- | :--- | :--- |
| overrideFeatureInstallOrder | array | Array consisting of the feature `id` of features in the order the user wants them to be installed.   |

#### (2) installsAfter

This property is defined in an individual feature's `devcontainer-feature.json` file by the feature author.  `installsAfter` allows an author to provide the tooling hints on loose dependencies between features.

After `overrideFeatureInstallOrder` is resolved, any remaining features that declare an `installsAfter` must be installed after the features declared in the property, provided that the features have also been declared in the `features` property.

| Property | Type | Description |
| :--- | :--- | :--- |
| installsAfter | array | Array consisting of the feature `id` that should be installed before the given feature   |

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

### Option Resolution Example

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

...output the following:
```
Version is 3.10
Pip? false
Optimize? true
```

### Implementation Notes

There are several things to keep in mind for an application that implements features:

- The order of execution of features is determined by the application, based on the `installAfter` property used by feature authors. It can be overridden by users if necessary with the `overrideFeatureInstallOrder` in `devcontainer.json`.
- Features are used to create an image that can be used to create a container or not.
- Parameters like `privileged`, `init` are included if just 1 feature requires them.
- Parameters like `capAdd`, `securityOp`  are concatenated.
- `containerEnv` is added before the feature is executed as `ENV` commands in the Dockerfile.
- Each feature script executes as its own layer to aid in caching and rebuilding.
