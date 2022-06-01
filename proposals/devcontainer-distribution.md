# Community creation of templates and features and their distribution.

This proposal is created to arrive at a schema where different communities and organizations can create features and devcontainer templates and make them discoverable to users.

The proposal centers on the following ideas:

- Discoverability for users both on the web (github.com) and on any tools that implement the [Dev container specification](../docs/specs/devcontainer-reference.md).
- Users can validate the origin of the template or feature that they want to use.
- Users can pin a particular version of a feature to allow consistent repeatable environments.

# Proposal

## Github Topics

This can be achieved with the use of [GitHub Topics](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/classifying-your-repository-with-topics), specifically the [devcontainers](https://github.com/topics/devcontainers) topic. Additionally the repositories can be marked with **devcontainers-templates** and/or **devcontainers-features** to mark specifically the contents of the repository.

Any UI application can leverage this search to provide a list of repositories that contain features or templates and present them for their users. The application would query the latest release and obtain the details to show them to the users.

An implementation of the spec can also use other mechanisms to speed up or improve the user experience while searching.

At the same time it should contain a way for users to directly reference a repository containing the features/templates that they want to use.

## Releases

As a distribution and storage mechanism we propose using GitHub releases. The releases would contain compressed files with the source code of the feature or template. Together with the release the corresponding metadata file would be included either **devcontainer-feature.json**, **devcontainer-template.json** or **devcontainer-collection.json**.

## Collections

Features and/or templates can be distributed both as a single unit or as a **collection** of them. In case they are released as a collection the release should include a **devcontainer-collection.json** file that groups the information of each feature/template contained.

| Property | Type | Description |
| :--- | :--- | :--- |
| publisher | string | The organization/user that publishes this group of templates/features. Must match with the repository user in github.|
| repository | string | The repository where this collection is contained.|
| version | string | Version information of this collection list.|
| features | array | The list of features that are contained in this collection. Same information as the metadata file for the feature.|
| templates | array | The list of templates contained in this collection. Same information as the metadata file for the definition.|

Its important to note the following about collections:

- It is recomended that they are created automatically as part of the release process.
- Ids would get the collection repository added to them. (e.g. for 'https://github.com/community/features' then 'community/features' to 'myfeature1' for a feature with that id)
- The links to the release files would be added.
- Date.
- Version.

## Templates specifics

By their nature templates are single use since they are just a group of files that are copied over to an end users repository and used to kickstart a **dev container** configuration.

Due to this the implementing application needs to ensure that the user is able to inspect all files included in the template and that no code is executed automatically until a specific action by a user demands it. 

## Features specifics

Since features contain code that can be executed multiple times by different users the security of the contents is important, for this we propose:

- Any application that implements the [Dev container spec](../docs/specs/devcontainer-reference.md) must provide a way for the user to add a feature to their `devcontainer.json` file in a way that the code can be inspected before it executes the first time.
- This action should create a **devcontainer.lock** file next to the main `devcontainer.json` file that contains the checksums of the particular release referenced.
- Users can commit this file to their repository so that the application can berify the checksum each time it rebuilds the container, throwing an error if they not match.
- If users want to update to a newer version they can either delete the **devcontainer.lock** file, delete the particular entry or reference a different version of it on their `devcontainer.json` file. If the version stored in both files doesn't match, the application should update the entry on the next build.
- If during a first build the **devcontainer.lock** file does not exist, it should be created as part of the build process if and when it succeeds.

The **devcontainer.lock** contains an array of:

| Property | Type | Description |
| :--- | :--- | :--- |
| id | string | The fully qualified id of the feature as defined [here](./devcontainer-features.md#devcontainerjson-properties).|
| options | string | The same options object passed to the feature in `devcontainer.json`|
| checksum | string | The checksum of the downloaded compressed release file.|

It is important to note that, like in `devcontainer.json`  features can appear more than one time and are assumed to be in the same order.