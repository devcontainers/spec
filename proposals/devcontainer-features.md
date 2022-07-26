# Dev container features reference

Dev container 'features' are self-contained units of installation code and development container configuration. Features are designed to install atop a wide-range of base container images. 
Features are generally designed to be installed on top of a subset of base container images (eg: debian-based images).

Feature metadata is captured by a `devcontainer-feature.json` file in the root folder of the feature.

## Folder Structure 

A feature is a self contained entity in a folder. A feature release would be a tar file that contains all files part of the feature.

```
+-- feature
|    +-- devcontainer-feature.json
|    +-- install.sh (default)
|    +-- (other files)
```

In case `devcontainer-feature.json`  does not include a reference for the lifecycle scripts the application will look for the default script names and will execute them if available.

In case there is intent to create a set of features that share code, it is possible to create a feature collection in the following way:

```
collectionFolder
+-- devcontainer-collection.json
+-- common (or similar)
|    +-- (library files)
+-- feature1
|    +-- devcontainer-feature.json
|    +-- install.sh
|    +-- (other files)
+-- feature2
(etc)
```


## devcontainer-feature.json properties

the `devcontainer-feature.json` file defines information about the feature to be used by any supporting tools and the way the feature will be executed.

The properties of the file are as follows:

| Property | Type | Description |
| :--- | :--- | :--- |
| id | string | Id of the feature/definition. The id should be unique in the context of the repository/published package where the feature exists. |
| version | string | The semantic version of the feature. |
| name | string | Name of the feature/definition. |
| description | string | Description of the feature/definition. |
| documentationURL | string | Url that points to the documentation of the feature. |
| licenseURL | string | Url that points to the license of the feature. |
| version | string | Version of the feature. |
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


Features are referenced in `devcontainer.json` under the top level `features` object. 

The properties are:
| Property | Type | Description |
| :--- | :--- | :--- |
| id | string | Reference to the particular feature to be included. |
| options | object | Type of the option .|

The `id` is the main reference point for how to find and download a particular feature. `id` can be defined in any of the following ways:

| Type | Description | Example |
| :--- | :--- | :--- |
| \<oci-registry\>/\<namespace\>/\<feature\>[:\<semantic-version\>] | Reference to feature in OCI registry(*) | ghcr.io/user/repo/go:1 |
| https://<..URI..>/myFeatures.tgz#{feature} | Direct HTTPS URI to a tarball. | https:github.com/user/repo/releases/myFeatures.tgz#go|
| ./{local-path}  -or-  | A relative to directory with a devcontainer-feature.json. | ./myGoFeature |

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

This propertry is defined in an individual feature's `devcontainer-feature.json` file by the feature author.  `installsAfter` allows an author to provide the tooling hints on loose dependencies between features.

After `overrideFeatureInstallOrder` is resolved, any remaining features that declare an `installsAfter` must be installed after the features declared in the property, provided that the features have also been declared in the `features` property.

| Property | Type | Description |
| :--- | :--- | :--- |
| installsAfter | array | Array consisting of the feature `id` that should be installed before the given feature   |


### Implementation Notes

There are several things to keep in mind for an application that implements features:

- The order of execution of features is determined by the application, based on the `installAfter` property used by feature authors. It can be overridden by users if necessary with the `overrideFeatureInstallOrder` in `devcontainer.json`.
- Features are used to create an image that can be used to create a container or not.
- Parameters like `privileged`, `init` are included if just 1 feature requires them.
- Parameters like `capAdd`, `securityOp`  are concatenated.
- `containerEnv` is added before the feature is executed as `ENV` commands in the Dockerfile.
- Each feature script executes as its own layer to aid in caching and rebuilding.
