# Dev container features reference

Dev container features provide a smooth path for customizing your container definitions. Features are self contained units of code meant to facilitate creation of custom images for application or development containers.

From a practical point of view, features are folders that contain units of code with different entrypoints for different lifecycle events.

Features can be defined by a `devcontainer-feature.json` file in the root folder of the feature. The file is optional for backwards compatibility but it is required for any new features being authored.

By default, features are installed in an order selected by the implementing tool. 

If any of the following properties are provided in the feature's `devcontainer-feature.json`, or the user's `devcontainer.json`, the order indicated by the propert(ies) are respected.

- `runsAfter` property defined as part of `devcontainer-feature.json`.
- `id`.

The tool uses the `runsAfter` property to intelligently manage this order and ensure that if there are relationships between the features, they are respected.

An end-user can explicitly provide an installation order for features given the  `overrideFeatureInstallOrder` property of `devcontainer.json`. 

All feature `id` provided in `overrideFeatureInstallOrder` must also exist in the `features` property of a user's `devcontainer.json`.

The provided features, indicated by `id`, will be installed in the specified order. Any remaining features in the features object that are not mentioned in the array will be installed in an undefined/implicit order, as determined as optimal by the tooling.

| Property | Type | Description |
| :--- | :--- | :--- |
| overrideFeatureInstallOrder | array | Array made of the Id's of the features in the order the user wants them to be installed. |


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
| name | string | Name of the feature/definition. |
| description | string | Description of the feature/definition. |
| documentationURL | string | Url that points to the documentation of the feature. |
| licenseURL | string | Url that points to the license of the feature. |
| version | string | Version of the feature. |
| keywords | array | List of keywords relevant to a user that would search for this definition/feature. |
| install.app | string | App to execute.|
| install.file | string | Parameters/script to execute (defaults to install.sh).|
| options | object | Possible options to be passed as environment variables to the execution of the scripts |
| containerEnv | object | A set of name value pairs that sets or overrides environment variables. |
| privileged | boolean | If priveleged mode is required by the feature. |
| init | boolean | If it's necesarry to run `init`. |
| capAdd | array | Additional capabilities needed by the feature. |
| securityOpt | array | Security options needed by the feature. |
| entrypoint | string | Set if the feature requires an entrypoint. |

Options

| Property | Type | Description |
| :--- | :--- | :--- |
| id | string | Id of the option. |
| id.type | string | Type of the option. |
| id.enum | string array | Possible values. |
| id.default | string | Default value for the option. |
| id.description | string | Description for the option. |

## devcontainer-collection.json properties

A feature collection file is a compilation of the `devcontainer-feature.json` files for each individual feature. It inlines all the information in each `devcontainer-feature.json` for each feature.

If the application finds a `devcontainer-collection.json` file in the root of a downloaded tar file, then it uses that file to find the particular feature that will be executed.

In addition to the list of features included, the file includes the following properties.

| Property | Type | Description |
| :--- | :--- | :--- |
| name | string | Name of the collection. |
| reference | string | Reference information of the repository and path where the code is stored. |
| version | string | Version of the code. |

In most cases, the `devcontainer-collection.json` file can be generated automatically at the moment of creating a release.

## devcontainer.json properties

Features are referenced in `devcontainer.json` , where the `features` tag consists of an object tag starting with the id of the feature and including the values of the options to pass to the feature itself.

The properties are:
| Property | Type | Description |
| :--- | :--- | :--- |
| options | object | Type of the option. |

The `id` is the main reference point for how to find and download a particular feature. `id` can be defined in any of the following ways:

| Type | Description |
| :--- | :--- |
| feature  | A simple name referencing a feature; it's included with the application used to execute `devcontainer.json`.|
| organization/repository/{feature or collectionl/feature}/@version | Reference to a particular release in a repository. |
| https://<../URI/..>/devcontainer-features.tgz#{optional feature} | Direct reference to a file download. |
| ./{local-path}  -or-  ../{local-path} | A path relative to devcontainer.json where a feature or feature collection can be found. |

## Authoring

Features can be authored in a number of languages, the most straightforward being bash scripts. If a feature is authored in a different language information about it should be included in the metadata so that users can make an informed choice about it.

Reference information about the application required to execute the feature should be included in `devcontainer-feature.json` in the metadata section.

Applications should default to `/bin/sh` for features that do not include this information.

If the feature is included in a folder as part of the repository that contains `devcontainer.json`, no other steps are necessary.

## Release

A release is created when the objective is to have other users use a feature.

A release consists of the following:

1.- Tar file with all the included files for the feature or feature collection.
2.- A copy of the `devcontainer-feature.json` or `devcontainer-collection.json` file that defines the contents of the tar file, with additional information added to validate it.

| Property | Type | Description |
| :--- | :--- | :--- |
| version | string | SemVer version of the release. |
| checksum | string | Checksum of the tar file. |

## Execution

There are several things to keep in mind for an application that implements features:

- The order of execution of features is determined by the application, based on the `installAfter` property used by feature authors. It can be overridden by users if necessary with the `overrideFeatureInstallOrder` in `devcontainer.json`.
- Features are used to create an image that can be used to create a container or not.
- Parameters like `privileged` and `init` are included if just 1 feature requires them.
- Parameters like `capAdd` and `securityOp`  are concatenated.
- `containerEnv` is added before the feature is executed as `ENV` commands in the Dockerfile.
- Each feature script executes as its own layer to aid in caching and rebuilding.