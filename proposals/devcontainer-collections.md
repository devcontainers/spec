# Proposal

The proposal is to arrive to an schema where different communities and organizations can create "collections" of features/definitions that they manage, support and present to users/

This collection data should have enough information so that different clients can search them and present them to users.

For this we propose the existence of two different files:
- Collection file: (devcontainer-collection.json) Groups of features/definitions that will be handled as the same origin (repository/publisher). Contains the metadata pertinent for each one, checksum and other information.
- Feature/Definition: (devcontainer-feature.json/devcontainer-definition.json) Information on a particular feature/definition required to add it to a collection. This file is optional and exists to support scenarios where one repository represents a collection and others represent each feature.

This files would ideally be created automatically by a process that automatically adds information like: checksums, signatures, download links, etc.

Clients would be able to select different sources for collections while also giving the option to users to add their own sources.

## Collection file (devcontainer-collection.json)

- Created as a release of a repository that either contains or represents groups of definitions/features from the same user/owner/publisher. (e.g. same organization in Github.)
- Contains metadata for each definition/feature, repository origin, download link, checksum or signing information.
- All referenced definitions/features would belong to the same user/organization. (e.g. download links would be the same organization in GitHub.)
- References exact releases of each definition/feature.

| Property | Type | Description |
| :--- | :--- | :--- |
| publisher | string | The organization/user that publishes this group of definitions/features. Must match with the repository user in github.|
| repository | string | The repository where this collection is contained.|
| version | string | Version information of this collection list.|
| features | array | The list of features that are contained in this collection. Same information as the metadata file for the feature.|
| definitions | array | The list of definitions contained in this collection. Same information as the metadata file for the definition.|

Creating the collections file would make some changes over the metadata of the features/definitions where they are generated.

- Ids would get the collection repository added to them. (e.g. for 'https://github.com/community/features' then 'community/features' to 'myfeature1' for a feature with that id)
- The links to the release files would be added.
- Date.

The same way creating the definitions/feature file would also be part of the release process for it so that the following metadata can be added:
- Version.
- Checksum.
- Signatures.
- Date.

## Definitions/features (devcontainer-feature.json/devcontainer-definition.json)

### Common Properties

| Property | Type | Description |
| :--- | :--- | :--- |
| id | string | Id of the feature/definition. The ids should be unique in the contest of the repository where they exist. |
| displayName | string | Name of the feature/definition |
| shortDescription | string | Optional one sentence description |
| description | string | Description of the feature/definition |
| repositoryPath | string | Path to the repository and folder that contains the root of the code for the feature/definition. |
| keywords | array | List of keywords relevant to a user that would search for this definition/feature. |
| version | string | Versioning information for the feature/definition. |
| files | array | List of links for the download files with checksums and optional signature information. |

### Definitions

| Property | Type | Description |
| :--- | :--- | :--- |
| type | string | singleContainer/orchestrated |
| categories | string | The categories where it belongs |
| architectures | string | Supported architectures |
| baseOs | string | The operating system in which the definition is based |
| options | array | List of user selected options at the moment the user selects them in a client application. |
| styles | array | Variations of the generated definition that contain different information. (e.g. with example code or data.) |

### Features

| Property | Type | Description |
|:--- |:--- |:--- |
| requires | array | List of features that are required for this feature to work. |

# IDs

TODO

# Search

Search would be implemented by client applications but the definitions themselves should include enough information for this to happen. Besides the typed properties defined above keywords could be:

- Purpose. WebApp, WebService, IDE extension, etc.
- Platform. Node, Ruby on Rails, .Net, etc.
- Language. TS, JS, C#, Java, etc.
- Other Apps. Databases, simulators, etc.
- Dependencies. Bare metal, Azure, AWS, etc.
- Organization. 

This keywords would allow users to add search information easily like versions or more details to refine searches. (e.g. Python2 vs Python3)

# Example

- User creates a group of features/definitions for his organization.
  - They are contained under multiple repositories under the "example" organization.
_ The user adds an action to create release of all the different features/definitions. (A reference implementation should be included with this specification. This action would take a base metadata file, create the release tar file, and create the end metadata file with the download links, checksums, etc.)
  - The action could run automatically with every commit and could run any tests that are defined for it.
  - Optionally the action could create a PR on the collection repository to update the version.
- User creates a repository under the same "example" organization to define the collection of features/ definitions.
  - This repository contains folders with files that point to the repositories that contain the features/definitions. (One folder for features, another for definitions.)
- The user adds an action (Reference implementation.) that reads those folders and grabs the metadata for each feature/definition it finds in those repositories, consolidates it into 1 file and creates a release with it.
  - The action could be triggered by any changes to the folders that contain the features. (e.g. from the resulting merge of the PR created before.)

# Remarks

- By maintaining the rule of the user origin of a collection and files we can expose that information to the user reliably.
- If the releases themselves are signed and the signature is valid we could also signal that information to the user.
- The contents of all JSON files (metadata and collections) are considered unsafe and should be validated rigorously by any clients.
- Trust is centered around the origin organization for a collection.
- Security is handled by the user policies (e.g. who can merge or push) of the containing repository.
- The community could define their own policies.