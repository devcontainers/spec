# Community contributions of templates and features

This proposal is created to arrive at a schema where different communities and organizations can create features and dev container templates and make them discoverable to users.

The proposal centers on the following ideas:

- Discoverability for users both on the web (github.com) and on any tools that implement the [Dev container specification](../docs/specs/devcontainer-reference.md).
- Users can validate the origin of the template or feature that they want to use.
- Users can pin a particular version of a feature to allow consistent, repeatable environments.

# Proposal

## Github Topics

A set of discoverable dev container-related repos can be achieved with the use of [GitHub Topics](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/classifying-your-repository-with-topics). You may explore repos that GitHub has automatically considered part of the [devcontainers](https://github.com/topics/devcontainers) topic. If you'd like your template or feature repo to be easily discoverable to the community, you may add the **devcontainers-templates** and/or **devcontainers-features** topic. You may read more on how to do this in the [topics guide](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/classifying-your-repository-with-topics).

Any UI application can leverage this search to provide a list of repositories that contain features or templates and present them to their users. The application would query the latest release of repos within the list and obtain their details to show them to the users.

An implementation of the spec can also use other mechanisms to speed up or improve the user experience while searching.

At the same time, it should include a way for users to directly reference a repository containing the features or templates they want to use.

## Releases

As a distribution and storage mechanism, we propose using GitHub releases. The releases would contain compressed files with the source code of the feature or template. Included with the release the corresponding metadata file would be either **devcontainer-feature.json**, **devcontainer-template.json** or **devcontainer-collection.json**.

## Collections

Features and/or templates can be distributed both as a single unit or as a **collection** of them. In case they are released as a collection, the release should include a **devcontainer-collection.json** file that groups the information of each feature or template contained.

| Property | Type | Description |
| :--- | :--- | :--- |
| publisher | string | The organization/user that publishes this group of templates/features. Must match with the repository user in GitHub.|
| repository | string | The repository where this collection is contained.|
| version | string | Version information of this collection list.|
| features | array | The list of features that are contained in this collection. Same information as the metadata file for the feature.|
| templates | array | The list of templates contained in this collection. Same information as the metadata file for the definition.|

It's important to note the following about collections:

- It is recomended that they are created automatically as part of the release process.
- When a **devcontainer-collection.json** file is created, the Id's of the features/templates should get the name of the collection added to them to aid in the search for the source. For example, in repository 'https://github.com/community/features' for a collection called 'collection', the resulting id would be 'community/features/collection/myFeature'.

## Templates specifics

By their nature, templates are single use since they are just a group of files that are copied over to an end user's repository and used to kickstart a **dev container** configuration.

Because of this, the implementing application needs to ensure that the user is able to inspect all files included in the template and that no code is executed automatically until a specific action by a user demands it. 

## Features specifics

Since features contain code that can be executed multiple times by different users, the security of the contents is important, and for this we propose:

- Any application that implements the [Dev container spec](../docs/specs/devcontainer-reference.md) must provide a way for the user to add a feature to their `devcontainer.json` file in a way that the code can be inspected before it executes the first time.
- This action should create a **devcontainer.lock** file next to the main `devcontainer.json` file that contains the checksums of the particular release referenced.
- Users can commit this file to their repository so that the application can berify the checksum each time it rebuilds the container, throwing an error if they do not match.
- If users want to update to a newer version, they can either delete the **devcontainer.lock** file, delete the particular entry, or reference a different version of it in their `devcontainer.json` file. If the version stored in both files doesn't match, the application should update the entry on the next build.
- If during a first build the **devcontainer.lock** file does not exist, it should be created as part of the build process if and when it succeeds.

The **devcontainer.lock** contains an array of:

| Property | Type | Description |
| :--- | :--- | :--- |
| id | string | The fully qualified id of the feature as defined [here](./devcontainer-features.md#devcontainerjson-properties).|
| options | string | The same options object passed to the feature in `devcontainer.json`.|
| checksum | string | The checksum of the downloaded compressed release file.|

The specification supports executing features multiple times with different options. This can be useful when a user wants to install multiple versions of sdk's, or simply execute whatever changes the feature makes to the operating system multiple times.