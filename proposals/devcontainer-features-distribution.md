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

For ease of authorship and maintenance, [1..n] features can share a single git repository.  This set of features is referred to as a collection, and will share the same [`devcontainer-collection.json`](#devcontainer-collection.json) file and 'namespace' (eg. `owner/repo`).

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

There are several supported ways to distribute features.  Distribution is handled by the implementing packaging tool.

A user references a distributed feature in a `devcontainer.json` as defined in ['referencing a feature'](./devcontainer-features.md#Referencing-a-feature).

### OCI Registry

An OCI registry that implements the [OCI Artifact Distribution Specification](https://github.com/opencontainers/distribution-spec) serves as the primary distribution mechanism for features.

Each packaged feature is pushed to the registry following the naming convention `<registry>/<namespace>/<id>[:version]`, where version is the major, minor, and patch version of the feature, according to the semver specification.

A custom media type `application/vnd.devcontainers` and `application/vnd.devcontainers.layer.v1+tar` are used as demonstrated below.


For example, the `go` feature in the `devcontainers/features` namespace at version `1.2.3` would be pushed to the ghcr.io OCI registry.  

_NOTE: The example below uses [`oras`](https://oras.land/) for demonstration purposes.  A supporting tool should directly implement the required functionality from the aforementioned OCI artifact distribution specification._
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
                             ./${ARTIFACT_PATH}:application/vnd.devcontainers.layer.v1+tar
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
                            ./devcontainer-collection.json.:application/vnd.devcontainers.layer.v1+json
```

### Directly Reference Tarball

A feature can be referenced directly in a `devcontainer.json` file by an HTTP URI that points to the tarball from the [package step](#packaging).
