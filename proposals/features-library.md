# Sharing code between Dev Container Features

ref: https://github.com/devcontainers/spec/issues/129

# Motivation

The current model for authoring Features encourages significant copy and paste between individual Features.  Most of the Features published under the `devcontainers/features` namespace contain approximately 100 lines of [identical helper functions](https://github.com/devcontainers/features/blob/main/src/azure-cli/install.sh#L29-L164) before the first line of the script is executed.  The process of maintaining these helper functions is tedious, error-prone, and leads to less readable Features.

Additionally, [members of the Feature authoring community have expressed interest in sharing code between Features](https://github.com/orgs/devcontainers/discussions/10).  Exploring the source code of self-published Features, it is evident that many authors have copy/pasted code from other Features.  This is a sign that there is a need for a more efficient way to share code between Features.

# Goal

Provide a generic and simple way to share code between Features.

# Proposals

## [Proposal A] Add: `include` property to `devcontainer-feature.json`

Add an `include` property to the `devcontainer-feature.json`.  This will be honored during the Feature [packaging stage](https://containers.dev/implementors/features-distribution/#packaging). 

The contents of the provided relative path is copied into the packaged Feature archive.  Files or directories are recursively copied into the archive following the same directory structure as in the source code.

The relative path must not escape the root directory - if this is attempted, the packaging stage will fail.

Property | Type | Description
--- | --- | ---
`include` | `string[]` | An array of paths relative to the directory containing this `devcontainer-feature.json`. If the element is a folder, it is copied recursively.  Must be prefixed with `.` or `..` to indicate the provided string is relative path.

### Example

```json
{
    "name": "featureA",
    "version": "0.0.1",
    "include": [
        "../../utils/",
        "../../company-welcome-message.txt"
    ]
}
```

## [Proposal B] Change: Structure of packaged Feature

Change the file structure of the packaged Feature to mirror the directory structure of the source code.  This will simplify the implementation of **Proposal A** and keep the relative paths consistent before and after packaging.

When running the packaging step, each Feature is packaged separately under `./src/<FeatureID>`.  Other included directories will be copied into the root of the packaged artifact as indicated in **Proposal A**.

An implementation should resolve the `devcontainer-feature.json` by checking for its presence in the following order of precedence:

 - `./src/<feature-id>/devcontainer-feature.json`
-  `devcontainer-feature.json`

From here forward, the proposed format will be used during the packaging step.  For backwards compatibility, the existing format (with a `devcontainer-feature.json` at the top level) will be supported.

### Example

Given this example repository containing a collection of Features.

```
.
├── images
│   ├── 1.png
│   ├── 2.png
├── company-welcome-message.txt
├── utils
|   ├── common.sh
│   └── helpers
│       ├── a.sh
│       └── ...
|
├── src
│   ├── featureA
│   │   ├── devcontainer-feature.json
│   │   ├── install.sh
│   │   └── ...
│   ├── featureB
│   │   ├── devcontainer-feature.json
│   │   └── install.sh
│   │   └── ...
|   ├── ...
│   │   ├── devcontainer-feature.json
│   │   └── install.sh
│   │   └── ...
├── ...
```

When packaging Feature A (shown above in **Proposal A**), the resulting published archive will be structured as follows:

```
.
├── company-welcome-message.txt
├── utils
|   ├── common.sh
│   └── helpers
│       ├── a.sh
│       └── ...
|
├── src
│   ├── featureA
│   │   ├── devcontainer-feature.json
│   │   ├── install.sh
│   │   └── ...
```

The packaging step recursively copies the contents of the `utils` folder into the resulting archive.  Additionally, the `company-welcome-message.txt` is packaged and distributed with the Feature.

Note that the `images` folder is not included in the packaged Feature.  This is because the `images` folder is not included in the `include` property of the `devcontainer-feature.json` file.

#### Using an included script in a Feature

The following example shows how a Feature could include a `utils` folder that contains a `common.sh` file.

```bash
# utils/common.sh
function common_function() {
   echo "Hello, '$1'"
}
```

```bash
# src/featureA/install.sh

# Include common functions
source "../../utils/common.sh"

# Use common function
common_function "devcontainers"
```

-----

# Possible Extensions

This section provides hints to possible future work that could extend the proposal above.  These are not part of the proposal, but are provided to help the community understand the potential future impact of this proposal.

## Distributing shared library Features

In an effort to reduce the need for authors to copy/paste auxillary scripts between Feature namespaces, provide the ability to publish "library" Features.  These Features would be published [following the Feature distribution spec](https://containers.dev/implementors/features-distribution/#distribution) under a new manifest type. 

A "library" can be included in a Feature's `include` property as an alternative to a relative path. By itself, a "library" Feature would not be installable.

```json

{
    "name": "featureA",
    "version": "0.0.1",
    "include": [
        "ghcr.io/devcontainers/features/utils:0.0.1"
    ]
}
