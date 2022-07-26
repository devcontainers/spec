# Community Contribution and Discovery of Dev Container Features

This specification defines a pattern where community members and organizations can author and self-publish [dev container 'features'](./devcontainer-features.md). 

Goals include:

- For community authors, a "self-served" mechanism for dev container feature publishing.
- For community members, a largely self-served mechanism for marking features as "discoverable" by [supporting tools](../docs/specs/supporting-tools.md).
- For users, ease of discoverability of features via [supporting tools](../docs/specs/supporting-tools.md).
- For users, the abilty to validate the origin of the asset and its integrity when compared to previous pulls. 
- For users, the ability for a user to pin to a particular version (absolute, or semantic version) of a feature to allow consistent, repeatable environments.

## Source Code

Features source code is stored in a git repository.

For ease of authorship and maintenance, [1..n] features can share a single git repository.  This set of features is referred to as feature collections, and will share the same [`devcontainer-collection.json`](#devcontainer-collection.json) file and namespace.

Source code for the set follows the example file structure below:

```
.
├── README.md
├── src
│   ├── dotnet
│   │   ├── devcontainer-feature.json
│   │   ├── install.sh
│   │   └── ...
|   ├
│   ├── go
│   │   ├── devcontainer-feature.json
│   │   └── install.sh
|   ├── ...
│   │   ├── devcontainer-feature.json
│   │   └── install.sh
├── test
│   ├── dotnet
│   │   ├── devcontainer-feature.json
│   │   └── ...
│   └── go
│   |   └── test.sh
|   ├── ...
│   │   └── test.sh
├── ...
```

Where `src` is a directory containing a sub-folder with the name of the feature (e.g. `dotnet` or `go`) with at least a file named `devcontainer-feature.json` that contains the feature metadata. Each feature sub-directory also contains an `install.sh` script that implementing tools will use to install the feature.  Each sub-directory should be named such that it matches the `id` field of the `devcontainer-feature.json`.  Other files can also be included in the sub-directory, and will be packaged along side the two required files.

Optionally, a mirrored `test` directory can be included with an accompanying `test.sh` script.  Implementing tools may use this to run tests against the given feature.

## Versioning

Each feature is individually [versioned according to the semver specification](https://semver.org/).  The `version` property in the respective `devcontainer-feature.json` file is parsed to determine if the feature should be republished.

Tooling that handles publishing features will not republish features if that exact version has already been published; however, tooling must republish major and minor versions in accordance with the semver specification.

## Packaging

Features are distributed as tarballs.  The tarball contains the entire contents of the feature sub-directory, including the `devcontainer-feature.json`, `install.sh`, and any other files in the directory.

The tarball is named `devcontainer-feature-<id>.tgz`, where `<id>` is the feature's `id` field.

A reference implementation for packaging and distributing features is provided as a GitHub Action (https://github.com/devcontainers/action).


### devcontainer-collection.json

The `devcontainer-collection.json` is an auto-generated metadata file.

| Property | Type | Description |
| :--- | :--- | :--- |
| sourceInformation | object | Metadata from the implementing packaging tool. |
| features | array | The list of features that are contained in this collection.|
<!-- | templates | array | The list of templates contained in this collection. Same information as the metadata file for the definition.| -->


Each features's `devcontainer-feature.json` metadata file is appended into the `features` top-level array.

## Distribution

There are several supported ways to distribute features.  With appropriate authentication a user references a distributed feature in a `devcontainer.json` as defined in ['referencing a feature'](./devcontainer-features.md#Referencing-a-feature).

### OCI Registry

An OCI registry that implements the [OCI Artifact Distribution Specification](https://github.com/opencontainers/distribution-spec) serves as the primary distribution mechanism for features.

Each packaged feature is pushed to the registry following the naming convention `<registry>/<namespace>/<id>[:version]`, where version is the major, minor, and patch version of the feature, according to the semver specification.

A custom media type `application/vnd.devcontainers` and `application/vnd.devcontainers.layer.v1+tar` are used as demonstrated below.

For example, the `go` feature in the `devcontainers/features` namespace at version `1.2.3` would be pushed to the ghcr.io OCI registry.
```bash
# ghcr.io/devcontainers/features/go:1 
REGISTRY=ghcr.io
NAMESPACE=devcontainers/features
FEATURE=go

ARTIFACT_PATH=devcontainer-feature-go.tgz

for VERSION in 1  1.2  1.2.3  latest
do
    oras push ${REGISTRY}/${NAMESPACE}/${FEATURE}:${VERSION} \
            --manifest-config /dev/null:application/vnd.devcontainers \
                             ./${ARTIFACT_PATH}:application/vnd.devcontainers.layer.v1+tar`;
done

```

`Namespace` is the globally identifiable name for the collection of features. (eg: `owner/repo` for the source code's git repository).

The auto-generated `devcontainer-collection.json` is pushed to the registry with the same `namespace` as above and no accompanying `feature` name. The collection file is always tagged as `latest`.

```bash
# ghcr.io/devcontainers/features
REGISTRY=ghcr.io
NAMESPACE=devcontainers/features

oras push ${REGISTRY}/${NAMESPACE}:latest \
        --manifest-config /dev/null:application/vnd.devcontainers \
                            ./devcontainer-collection.json.:application/vnd.devcontainers.layer.v1+json`;
```

### Directly reference tarball

A feature can be referenced directly in a `devcontainer.json` file by an HTTP URI that points to the tarball from the [package step](#packaging).

<!-- 
## Github Topics

A set of discoverable dev container-related repos can be achieved with the use of [GitHub Topics](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/classifying-your-repository-with-topics). You may explore repos that GitHub has automatically considered part of the [devcontainers](https://github.com/topics/devcontainers) topic. If you'd like your template or feature repo to be easily discoverable to the community, you may add the **devcontainers-templates** and/or **devcontainers-features** topic. You may read more on how to do this in the [topics guide](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/classifying-your-repository-with-topics).

Any UI application can leverage this search to provide a list of repositories that contain features or templates and present them to their users. The application would query the latest release of repos within the list and obtain their details to show them to the users.

An implementation of the spec can also use other mechanisms to speed up or improve the user experience while searching.

At the same time, it should include a way for users to directly reference a repository containing the features or templates they want to use.

## Releases

As a distribution and storage mechanism, we propose using GitHub releases. The releases would contain compressed files with the source code of the feature or template. Included with the release the corresponding metadata file would be either **devcontainer-feature.json**, **devcontainer-template.json** or **devcontainer-collection.json**.

## Collections

Features and/or templates can be distributed both as a single unit or as a **collection** of them. In case they are released as a collection, the release should include a **devcontainer-collection.json** file that groups the information of each feature or template contained.

| Property | Type | Description |
| :--- | :--- | :--- |
| publisher | string | The organization/user that publishes this group of templates/features. Must match with the repository user in GitHub.|
| repository | string | The repository where this collection is contained.|
| version | string | Version information of this collection list.|
| features | array | The list of features that are contained in this collection. Same information as the metadata file for the feature.|
| templates | array | The list of templates contained in this collection. Same information as the metadata file for the definition.|

It's important to note the following about collections:

- It is recomended that the **devcontainer-collection.json** file is created automatically as part of the release process.
- The compressed release file should contain all the folders of the features/templates included in it.
- The names of the folders should coincide with the id of the feature/template itself.
- When a **devcontainer-collection.json** file is created, the Id's of the features/templatescontained in it should get the name of the collection added to them to aid in the search for the source. For example, in repository 'https://github.com/community/features' for a collection called 'collection', the resulting id would be 'community/features/collection/myFeature'.

## Templates specifics

By their nature, templates are single use since they are just a group of files that are copied over to an end user's repository and used to kickstart a **dev container** configuration.

Because of this, the implementing application needs to ensure that the user is able to inspect all files included in the template and that no code is executed automatically until a specific action by a user demands it. 

## Features specifics

Since features contain code that can be executed multiple times by different users, the security of the contents is important, and for this we propose:

- Any application that implements the [Dev container spec](../docs/specs/devcontainer-reference.md) must provide a way for the user to add a feature to their `devcontainer.json` file in a way that the code can be inspected before it executes the first time.
- This action should create a **devcontainer.lock** file next to the main `devcontainer.json` file that contains the checksums of the particular release referenced.
- Users can commit this file to their repository so that the application can berify the checksum each time it rebuilds the container, throwing an error if they do not match.
- If users want to update to a newer version, they can either delete the **devcontainer.lock** file, delete the particular entry, or reference a different version of it in their `devcontainer.json` file. If the version stored in both files doesn't match, the application should update the entry on the next build.
- If during a first build the **devcontainer.lock** file does not exist, it should be created as part of the build process if and when it succeeds.

The **devcontainer.lock** contains an array of:

| Property | Type | Description |
| :--- | :--- | :--- |
| id | string | The fully qualified id of the feature as defined [here](./devcontainer-features.md#devcontainerjson-properties).|
| options | string | The same options object passed to the feature in `devcontainer.json`.|
| checksum | string | The checksum of the downloaded compressed release file.|

The specification supports executing features multiple times with different options. This can be useful when a user wants to install multiple versions of sdk's, or simply execute whatever changes the feature makes to the operating system multiple times. -->