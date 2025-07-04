# User Env Variables for Features

This has now been implemented:
* Discussion issue: https://github.com/devcontainers/spec/issues/91
* CLI PR: https://github.com/devcontainers/cli/pull/228

Below is the original proposal.

## Goal

Feature scripts run as the `root` user and sometimes need to know which user account the dev container will be used with.

(The dev container user can be configured through the `remoteUser` property in `devcontainer.json`. If that is not set, the container user will be used.)

## Proposal

Pass `_REMOTE_USER` and `_CONTAINER_USER` environment variables to the features scripts with `_CONTAINER_USER` being the container's user and `_REMOTE_USER` being the configured `remoteUser`. If no `remoteUser` is configured, `_REMOTE_USER` is set to the same value as `_CONTAINER_USER`.

Additionally the home folders of the two users are passed to the feature scripts as `_REMOTE_USER_HOME` and `_CONTAINER_USER_HOME` environment variables.

## Notes

- The container user can be set with `containerUser` in `devcontainer.json` and image metadata, `user` in the docker-compose.yml, `USER` in the Dockerfile and can be passed down from the base image.
