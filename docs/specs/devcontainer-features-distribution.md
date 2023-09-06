# Distribution and Discovery of Dev Container Features

**TL;DR Check out the [quick start repository](https://github.com/devcontainers/feature-template) to get started on distributing your own Dev Container Features.**

This specification defines a pattern where community members and organizations can author and self-publish [Dev Container Features](./devcontainer-features.md). 

Goals include:

- For Feature authors, create a "self-service" way to publish a Feature, either publicly or privately, that is not centrally controlled.
- For users, provide the ability to validate the integrity of fetched Feature assets. 
- For users, provide the ability to pin to a particular version (absolute, or semantic version) of a Feature to allow for consistent, repeatable environments.
- Provide the ability to standardize publishing such that [supporting tools](../docs/specs/supporting-tools.md) may implement their own mechanism to aid Feature discoverability as they see fit.

> **Tip:** This section covers details on the Features specification. If you are looking for summarized information on creating your own Features, see the [template](https://github.com/devcontainers/feature-template) and [core Features](https://github.com/devcontainers/features) repositories.

## Source code

Features source code is stored in a git repository.

For ease of authorship and maintenance, [1..n] features can share a single git repository. This set of features is referred to as a "collection," and will share the same [`devcontainer-collection.json`](#devcontainer-collection.json) file and "namespace" (eg. `<owner>/<repo>`).

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
│   │   ├── test.sh
│   │   └── ...
│   └── go
│   |   └── test.sh
|   ├── ...
│   │   └── test.sh
├── ...
```

...where `src` is a directory containing a sub-folder with the name of the feature (e.g. `src/dotnet` or `src/go`) with at least a file named `devcontainer-feature.json` that contains the feature metadata, and an `install.sh` script that implementing tools will use as the entrypoint to install the feature.

Each sub-directory should be named such that it matches the `id` field of the `devcontainer-feature.json`.  Other files can also be included in the feature's sub-directory, and will be included during the [packaging step](#packaging) alongside the two required files.  Any files that are not part of the feature's sub-directory (e.g. outside of `src/dotnet`) will not included in the [packaging step](#packaging).

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
| `sourceInformation` | object | Metadata from the implementing packaging tool. |
| `features` | array | The list of features that are contained in this collection.|

Each features's `devcontainer-feature.json` metadata file is appended into the `features` top-level array.

## Distribution

There are several supported ways to distribute Features. Distribution is handled by the implementing packaging tool such as the [Dev Container CLI](https://github.com/devcontainers/cli) or [Dev Container Publish GitHub Action](https://github.com/marketplace/actions/dev-container-publish). See the [quick start repository](https://github.com/devcontainers/feature-template) for a full working example.

> Feature identifiers should be provided lowercase by an author. If not, an implementing packaging tool should normalize the identifier into lowercase for any internal representation.

A user references a distributed feature in a `devcontainer.json` as defined in ['referencing a feature'](./devcontainer-features.md#Referencing-a-feature).

### OCI Registry

An OCI registry that implements the [OCI Artifact Distribution Specification](https://github.com/opencontainers/distribution-spec) serves as the primary distribution mechanism for Features.

Each packaged feature is pushed to the registry following the naming convention `<registry>/<namespace>/<id>[:version]`, where version is the major, minor, and patch version of the feature, according to the semver specification.

> **Note:** The `namespace` is a unique identifier for the collection of features.  There are no strict rules for the `namespace`; however, one pattern is to set `namespace` equal to source repository's `<owner>/<repo>`.  The `namespace` should be lowercase, following [the regex provided in the OCI specification](https://github.com/opencontainers/distribution-spec/blob/main/spec.md#pulling-manifests).

A custom media type `application/vnd.devcontainers` and `application/vnd.devcontainers.layer.v1+tar` are used as demonstrated below.

For example, the `go` feature in the `devcontainers/features` namespace at version `1.2.3` would be pushed to the ghcr.io OCI registry.  

> **Note:** The example below uses [`oras`](https://oras.land/) for demonstration purposes.  A supporting tool should directly implement the required functionality from the aforementioned OCI artifact distribution specification.

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

The "namespace" is the globally identifiable name for the collection of features. (eg: `owner/repo` for the source code's git repository).

The auto-generated `devcontainer-collection.json` is pushed to the registry with the same `namespace` as above and no accompanying `feature` name. The collection file is always tagged as `latest`.

```bash
# ghcr.io/devcontainers/features
REGISTRY=ghcr.io
NAMESPACE=devcontainers/features

oras push ${REGISTRY}/${NAMESPACE}:latest \
        --manifest-config /dev/null:application/vnd.devcontainers \
                            ./devcontainer-collection.json:application/vnd.devcontainers.collection.layer.v1+json
```

Additionally, an [annotation](https://github.com/opencontainers/image-spec/blob/main/annotations.md) named `dev.containers.metadata` should be populated on the manifest when published by an implementing tool.  This annotation is the escaped JSON object of the entire `devcontainer-feature.json` as it appears during the [packaging stage](https://containers.dev/implementors/features-distribution/#packaging).  

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

### Directly referencing a tarball

A Feature can be referenced directly in a user's [`devcontainer.json`](/docs/specs/devcontainer-reference.md#devcontainerjson) file by HTTPS URI that points to the tarball from the [package step](#packaging).

The `.tgz` archive file must be named `devcontainer-feature-<featureId>.tgz`.

### Locally referenced Features

Instead of publishing a Feature to an OCI registry, a Feature's source code may be referenced from a local folder. Locally referencing a Feature may be useful when first authoring a Feature.

A local Feature is referenced in the devcontainer's `feature` object **relative to the folder containing the project's `devcontainer.json`**.

Additional constraints exists when including local Features in a project:

* The project must have a `.devcontainer/` folder at the root of the [**project workspace folder**](/docs/specs/devcontainer-reference.md#project-workspace-folder).

* A local Feature's source code **must** be contained within a sub-folder of the `.devcontainer/ folder`.

* The sub-folder name **must** match the Feature's `id` field.

* A local Feature may **not** be referenced by absolute path.

* The local Feature's sub-folder **must** contain at least a `devcontainer-feature.json` file and `install.sh` entrypoint script, mirroring the [previously outlined file structure](#Source-code).


The relative path is provided using unix-style path syntax (eg `./myFeature`) regardless of the host operating system.

An example project is illustrated below:

```
.
├── .devcontainer/
│   ├── localFeatureA/
│   │   ├── devcontainer-feature.json
│   │   ├── install.sh
│   │   └── ...
│   ├── localFeatureB/
│   │   ├── devcontainer-feature.json
│   │   ├── install.sh
│   │   └── ...
│   ├── devcontainer.json
```

##### devcontainer.json
```jsonc
{
        // ...
        "features": {
                "./localFeatureA": {},
                "./localFeatureB": {}
        }

}
```
