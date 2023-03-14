# Sharing code between Dev Container Features

ref: https://github.com/devcontainers/spec/issues/129

# Motivation

The current model for authoring Features encourages significant copy and paste between individual Features.  Most of the Features published under the `devcontainers/features` namespace contain approximately 100 lines of [identifical helper functions](https://github.com/devcontainers/features/blob/main/src/azure-cli/install.sh#L29-L164) before the first line of the script is executed.  The process of maintaining these helper functions is tedious, error-prone, and leads to less readable Features.

Additionally, [members of the Feature authoring community have expressed interest in sharing code between Features](https://github.com/orgs/devcontainers/discussions/10).  Exploring the source code of self-published Features, it is evident that many authors have copy/pasted code from other Features.  This is a sign that there is a need for a more efficient way to share code between Features.

# Goal

Provide a generic and efficient way to share code between Features.

# Proposal

Add an `include` property to the `devcontainer-feature.json`.  This will be honored during the Feature [packaging stage](https://containers.dev/implementors/features-distribution/#packaging). The contents of the provided relative path is copied into the packaged Feature archive.

Property | Type | Description
--- | --- | ---
`include` | `string[]` | An array of relative paths to package with the Feature. Paths are relative to root directory when packaging (often the git root directory).  If the element is a folder, it is copied recursively.  Must be prefixed with `.` or `..` to resolve the relative path.

## Example

```json
{
    "name": "my-feature",
    "version": "0.0.1",
    "include": [
        "./utils/",
    ]
}
```

The preceeding example will recursively copy the contents of the the `utils` folder into the temporary folder used to package the Feature.  The `utils` folder will be included with the published Features.

This functionality will enable sharing a common set of helper functions between Features.  The following example shows how a Feature could include a `utils` folder that contains a `common.sh` file.

```bash
# utils/common.sh
function common_function() {
   echo "Hello, '$1'"
}
```

```bash
# my-feature/install.sh

# Include common functions
source "./utils/common.sh"

# Use common function
common_function "devcontainers"
```

-----

# Possible Extensions

This section provides hints to possible future work that could extend the proposal above.  These are not part of the proposal, but are provided to help the community understand the potential future impact of this proposal.

## Distributing shared library Features

In an effort to reduce the need for authors to copy/paste auxillary scripts between Feature namespaces, provide the ability to publish "library" Features.  These Features would be published [following the Feature distribution spec](https://containers.dev/implementors/features-distribution/#distribution) under a new manifest type. 

A "library" can be included in a Feature's `include` property as an alterative to a relative path. By themselves, a "library" Feature would not be installable.

```json

{
    "name": "my-feature",
    "version": "0.0.1",
    "include": [
        "ghcr.io/devcontainers/features/utils:0.0.1"
    ]
}
