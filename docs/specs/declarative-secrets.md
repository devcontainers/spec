# Declarative Secrets

This has now been implemented:
* Discussion issue: https://github.com/devcontainers/spec/issues/198, https://github.com/devcontainers/spec/issues/216
* Schema PR: https://github.com/devcontainers/spec/pull/303

Below is the original proposal.

## Motivation

Various projects exist in the wild that require various secrets for them to run properly. Examples include:

- https://github.com/lostintangent/codespaces-langchain
- https://github.com/openai/openai-quickstart-python
- https://github.com/openai/openai-quickstart-node

Today these projects have to include specific instructions e.g. in their README telling users where to procure what they need to run and then how to set it up as a secret or add it as an environment variable. This currently acts as an impediment to adoption and promotion of dev containers for these projects.

## Goal

Simplify using dev containers for these kinds of projects by supporting secrets as a first-class part of the dev container creation flow.

## Proposal

Add an optional `secrets` property to the `devContainer.base.schema.json`. This will be used to declare the secrets needed within the dev container.

Property | Type | Description
--- | --- | ---
`secrets` | `object` | The keys of this object are the names of the secrets that are in use. Keys should be [valid Linux environment variable names](https://pubs.opengroup.org/onlinepubs/000095399/basedefs/xbd_chap08.html).

The `secrets` property contains a map of secret names and details about those secrets. These `secrets` are recommended to the user but are not _required_ for creation. All properties are optional.

Property | Type | Description
--- | --- | ---
`description` | `string` | A brief description of the secret.
`documentationUrl` | `string` | A URL pointing to where the user can obtain more information about the secret. For example where to find docs on provisioning an API key for a service.

## Example

```json
{
    "image": "mcr.microsoft.com/devcontainers/base:bullseye",
    "secrets": {
        "CHAT_GPT_API_KEY": {
            "description": "I'm your super cool API key for ChatGPT.",
            "documentationUrl": "https://openai.com/api/"
        },
        "STABLE_DIFFUSION_API_KEY": {}
    }
}
```

This example would declare that this dev container wants the user to provide two secrets. The `CHAT_GPT_API_KEY` secret also provides some additional metadata that clients can use in displaying a user experience to provide that secret. The `STABLE_DIFFUSION_API_KEY` skips that metadata.

Implementations _may_ inject these secrets into the container as environment variables.

## Possible Future Extensions

These are **out of scope** for this proposal but the following has been discussed in relation:

1. Inclusion of a `type` field per secret. The default, which we are assuming and leveraging here, would be `"environmentVariable"`. This would include an option to specify `"reference"` and then reference the provided secrets in other parts of the dev container.
2. Prescriptions of _how_ the secrets are injected into the dev container.