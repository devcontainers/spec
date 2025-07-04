# Lockfiles

This has now been implemented:
* Discussion issue: https://github.com/devcontainers/spec/issues/236
* CLI PR: https://github.com/devcontainers/cli/issues/564

Below is the original proposal.

## Goal

Introduce a lockfile that records the exact version, download information and checksums for each feature listed in `devcontainer.json`.

This will allow for:
- Improved reproducibility of image builds (installing "latest" of a tool will still have different outcomes as the tool publishes new releases).
- Improved cachability of image builds (image cache checksums will remain stable when the lockfile pins a feature to a particular version).
- Improved security by detecting when a feature's release artifact changes after its checksum was first recorded in the lockfile ("trust on first use").

## Proposal

(The following is inspired by NPM's `package-lock.json` and Yarn's `yarn.lock`.)

Each feature is recorded with the identifier and version it is referred to in `devcontainer.json` as its key and the following properties as its values:
- `resolved`:
    - OCI feature: A qualified feature id with the sha256 hash (not the version number).
    - tarball feature: The `https:` URL used to download the feature.
- `version`: The full version number.
- `integrity`: `sha256:` followed by the hex encoded SHA-256 checksum of the download artifact.
- `dependsOn`: An array of feature identifiers equal to the feature's `dependsOn` keys in its `devcontainer-feature.json` including any version or checksum suffix. If the array is empty, this property can be omitted. For every feature listed here, the lockfile will also have a feature record.

The feature identifiers recorded as keys and in the `dependsOn` arrays are converted to lowercase to simplify lookups and comparisons. Note that the `devcontainer.json` and `devcontainer-features.json` may refer to the same features with different casing because these identifiers are not case-senstive.

Local features and the deprecated GitHub releases features are not recorded in the lockfile.

The lockfile is named `devcontainer-lock.json`, is located in the same folder as the `devcontainer.json` and contains a JSON object with a `"features"` property holding the above keys.

Example:

```jsonc
{
    "features": {
        "ghcr.io/devcontainers/features/node:1": {
            "version": "1.0.4",
            "resolved": "ghcr.io/devcontainers/features/node@sha256:567d704b3f4d3eca3acee51ded7c460a8395436d135d53d1175fb565daff42b8",
            "integrity": "sha256:567d704b3f4d3eca3acee51ded7c460a8395436d135d53d1175fb565daff42b8"
        },
        "https://mycustomdomain.com/devcontainer-feature-myfeature.tgz": {
            "version": "1.2.3",
            "resolved": "https://mycustomdomain.com/devcontainer-feature-myfeature.tgz",
            "integrity": "sha256:567d704b3f4d3eca3acee51ded7c460a8395436d135d53d1175fb565daff42b8"
        }
    }
}
```

Example with dependency:

```jsonc
{
  "features": {
    "ghcr.io/codspace/dependson/a": {
      "version": "1.2.1",
      "resolved": "ghcr.io/codspace/dependson/a@sha256:932027ef71da186210e6ceb3294c3459caaf6b548d2b547d5d26be3fc4b2264a",
      "integrity": "sha256:932027ef71da186210e6ceb3294c3459caaf6b548d2b547d5d26be3fc4b2264a",
      "dependsOn": [
        "ghcr.io/codspace/dependson/e:2"
      ]
    },
    "ghcr.io/codspace/dependson/e:2": {
      "version": "2.3.4",
      "resolved": "ghcr.io/codspace/dependson/e@sha256:9f36f159c70f8bebff57f341904b030733adb17ef12a5d58d4b3d89b2a6c7d5a",
      "integrity": "sha256:9f36f159c70f8bebff57f341904b030733adb17ef12a5d58d4b3d89b2a6c7d5a"
    }
  }
}
```
