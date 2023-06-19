# How to Contribute to the Dev Container Specification

We're excited for your contributions to the dev container specification! This document outlines how you can get involved. 

## Contribution approaches

If you'd like to contribute a change or addition to the spec, you may follow the guidance below:
- Propose the change via an [issue](https://github.com/microsoft/dev-container-spec/issues) in this repository. Try to get early feedback before spending too much effort formalizing it.
- More formally document the proposed change in terms of properties and their semantics. Look to format your proposal like our [devcontainer.json reference](https://aka.ms/devcontainer.json), which is a JSON with Comments (jsonc) format.

Here is a sample:

| Property | Type | Description |
|----------|------|-------------|
| `image` | string | **Required** when [using an image](/docs/remote/create-dev-container.md#using-an-image-or-dockerfile). The name of an image in a container registry ([DockerHub](https://hub.docker.com), [GitHub Container Registry](https://docs.github.com/packages/guides/about-github-container-registry), [Azure Container Registry](https://azure.microsoft.com/services/container-registry/)) that VS Code and other `devcontainer.json` supporting services / tools should use to create the dev container. |

- PRs to the [schema](https://github.com/devcontainers/spec/blob/main/schemas/devContainer.base.schema.json), i.e code or shell scripts demonstrating approaches for implementation.

Once there is discussion on your proposal, please also open and link a PR to update the [devcontainer.json reference doc](https://github.com/microsoft/vscode-docs/blob/main/docs/remote/devcontainerjson-reference.md). When your proposal is merged, the docs will be kept up-to-date with the latest spec.

### Contributing tool-specific support

Tool-specific properties are contained in namespaces in the `"customizations"` property. For instance, VS Code specific properties are formated as:

```bash
// Configure tool-specific properties.
"customizations": {
     // Configure properties specific to VS Code.
     "vscode": {
          // Set *default* container specific settings.json values on container create.
          "settings": {},
			
          // Additional VS Code specific properties...
     }
},
```

You may propose adding a new namespace for a specific tool, and any properties specific to that tool.

### GitHub Discussions
If you'd like to discuss the spec, such as asking questions, providing feedback, or engaging on how your team may use or contribute to dev containers, please check out the [GitHub Discussions](https://github.com/devcontainers/spec/discussions) in this repo. This is a great opportunity to connect with the community and maintainers of this project, without the requirement of contributing a change to the actual spec (which we see more in issues and PRs).

## Review process

We use the following [labels](https://github.com/microsoft/dev-container-spec/labels):

- `proposal`: Issues under discussion, still collecting feedback.
- `finalization`: Proposals we intend to make part of the spec.

[Milestones](https://github.com/microsoft/dev-container-spec/milestones) use a "month year" pattern (i.e. January 2022). If a finalized proposal is added to a milestone, it is intended to be merged during that milestone.
