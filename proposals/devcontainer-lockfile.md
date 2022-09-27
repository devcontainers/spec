## Goal

Introduce a lockfile that records the exact version, download information and checksums for each feature listed in the devcontainer.json.

This will allow for:
- Improved reproducibility of image builds (installing "latest" of a tool will still have different outcomes as the tool publishes new releases).
- Improved cachability of image builds (image cache checksums will remain stable when the lockfile pins a feature to a particular version).
- Improved security by detecting when a feature's release artifact changes after its checksum was first recorded in the lockfile ("trust on first use").

## Proposal

(The following is inspired by NPM's `package-lock.json` and Yarn's `yarn.lock`.)

Each feature is recorded with the identifier and version it is referred to in the devcontainer.json as its key and the following properties as its values:
- `resolved`:
    - OCI feature: A qualified feature id with the sha256 hash (not the version number).
    - tarball feature: The `https:` URL used to download the feature.
- `version`: The full version number.
- `integrity`: `sha256:` followed by the hex encoded SHA-256 checksum of the download artifact.

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
        "https://mycustomdomain.com/myscript-1.2.3.tgz": {
            "version": "1.2.3",
            "resolved": "https://mycustomdomain.com/myscript-1.2.3.tgz",
            "integrity": "sha256:567d704b3f4d3eca3acee51ded7c460a8395436d135d53d1175fb565daff42b8"
        }
    }
}
```
