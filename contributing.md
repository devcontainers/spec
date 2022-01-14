# How to Contribute to the Dev Container Specification

We're excited for your contributions to the dev container specification! This document outlines how you can get involved. 

## Contribution approaches

- Propose the change via an [issue](https://github.com/microsoft/dev-container-spec/issues) in this repository. Try to get early feedback before spending too much effort formalizing it.
- More formally document the proposed change in terms of properties and their semantics. Look to format your proposal like our [devcontainer.json reference](https://aka.ms/devcontainer.json).

Here is a sample:

| Property | Type | Description |
|----------|------|-------------|
| `image` | string | **Required** when [using an image](/docs/remote/create-dev-container.md#using-an-image-or-dockerfile). The name of an image in a container registry ([DockerHub](https://hub.docker.com), [GitHub Container Registry](https://docs.github.com/packages/guides/about-github-container-registry), [Azure Container Registry](https://azure.microsoft.com/services/container-registry/)) that VS Code and other `devcontainer.json` supporting services / tools should use to create the dev container. |

- PRs to the [schema](https://github.com/microsoft/vscode/blob/main/extensions/configuration-editing/schemas/devContainer.schema.src.json), i.e code or shell scripts demonstrating approaches for implementation.

Once there is discussion on your proposal, please also open and link a PR to update the [devcontainer.json reference doc](https://github.com/microsoft/vscode-docs/blob/main/docs/remote/devcontainerjson-reference.md). When your proposal is merged, the docs will be kept up-to-date with the latest spec.

## Review process

We use the following [labels](https://github.com/microsoft/dev-container-spec/labels):

- `proposal`: Issues under discussion, still collecting feedback.
- `finalization`: Proposals we intend to make part of the spec.

[Milestones](https://github.com/microsoft/dev-container-spec/milestones) use a "month year" pattern (i.e. January 2022). If a finalized proposal is added to a milestone, it is intended to be merged during that milestone.
