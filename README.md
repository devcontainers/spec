# Development Containers

This repository is for the Development Container Specification. You can check out the spec on our website https://containers.dev/ too.

A development container allows you to use a container as a full-featured development environment. It can be used to run an application, to separate tools, libraries, or runtimes needed for working with a codebase, and to aid in continuous integration and testing.

The Development Container Specification seeks to find ways to enrich existing formats with common development specific settings, tools, and configuration while still providing a simplified, un-orchestrated single container option – so that they can be used as coding environments or for continuous integration and testing.

![Stages of container-based development, from development to deployment](images/dev-container-stages.png)

The first format in the specification, `devcontainer.json`, was born out of necessity. It is a structured JSON with Comments (jsonc) metadata format that tools can use to store any needed configuration required to develop inside of local or cloud-based containerized coding. 

We envision that this same structured data can be embedded in images and other formats – all while retaining a common object model for consistent processing. For example, some of this same metadata can be added to a `devcontainer.metadata` image label to tie settings directly to a container image.

Beyond repeatable setup, these same development containers provide consistency to avoid environment specific problems across developers and centralized build and test automation services. You can use the [open-source CLI reference implementation](https://github.com/devcontainers/cli) either directly or integrated into product experiences to use the structured metadata to deliver these benefits. It currently supports integrating with Docker Compose and a simplified, un-orchestrated single container option – so that they can be used as coding environments or for continuous integration and testing.

A GitHub Action and an Azure DevOps Task are available in [devcontainers/ci](https://github.com/devcontainers/ci) for running a repository's dev container in continuous integration (CI) builds. This allows you to reuse the same setup that you are using for local development to also build and test your code in CI.

### Spec content

You may review the specification in the [docs/specs folder](https://github.com/devcontainers/spec/tree/main/docs/specs) of this repo.

You may also review active proposals in the [proposals folder](https://github.com/devcontainers/spec/tree/main/proposals).

Images used in this repo will be contained in the [images folder](/images). The icon for the [dev container GitHub org](https://github.com/devcontainers) is from the [Fluent icon library](https://github.com/microsoft/fluentui-system-icons/blob/master/assets/Cube/SVG/ic_fluent_cube_32_filled.svg).

## Contributing and Feedback

If you are interested in contributing, please check out the [How to Contribute](contributing.md) document, open an issue, or [join our community Slack channel](https://aka.ms/dev-container-community).

Please report issues in the following repositories:

- Spec-maintained Features and Templates: [devcontainers/features](https://github.com/devcontainers/features), [devcontainers/templates](https://github.com/devcontainers/templates)
- CLI reference implementation and non-spec related feature requests: [devcontainers/cli](https://github.com/devcontainers/cli)
- GitHub Action and Azure DevOps Task: [devcontainers/ci](https://github.com/devcontainers/ci)

# License

License for this repository:

Copyright © Microsoft Corporation All rights reserved.<br />
Creative Commons Attribution 4.0 License (International): https://creativecommons.org/licenses/by/4.0/legalcode
