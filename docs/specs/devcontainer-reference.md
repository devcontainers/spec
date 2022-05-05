# Devcontainers specificaction

The purpose of this specification is to define a file format and accompanying tools required to generate repeatable **development containers**.

The focus of the devcontainer specification is to describe how to enrich a container for the purposes of development rather than acting as a multi-container orchestrator format. Instead, container orchestrator formats can be referenced when needed to manage multiple containers and their lifecycle. Today, `devcontainer.json` includes scenario specific properties for working without a container orchestrator (by directly referencing an image or Dockerfile) and for using Docker Compose as a simple multi-container orchestrator.

A **development container** is defined as the set of logically related containers (as defined by the underlying orchestrator format) and the different lifecycle events that operate on them for this environments to be useful for development. Additionally, tools that want to implement this specification should provide a set of features/commands that give more flexibility to users and allow this **development containers** to scale to large development groups. Even though a **development container** can consist of multiple environments, most operations deal with the **main** container defined in `devcontainer.json`.

# Application Model

**Development containers** allow one to define a repeatable development environment for a team that is related to the execution environment of the application itself. In a way a **development container** can be a superset of all the required tools for the application being developed adding any tools required for compiling, debugging, diagnosing and testing said application.

In this intent a **development container** is composed of a deterministic definition on a `devcontainer.json` file that creates containers under the control of the user.

The `devcontainer.json` specification contains diferent configuration options related to the underlying orchestrator format and its not expected that all implementations support all of them. At the same time this specification leaves space for further development and implementation of other orchestrator mechanisms and file formats.

The application model consists of actions defined at different points in the lifecycle of the **development container** that allow users to control all aspects of how they are created and managed. Similarly this specification allows users to have other mechanisms to generate the different images required by their application and just reference them in `devcontainer.json`.

The **development containers** format is primarily tasked with the setup of a container or group of containers required for an application while focusing the configuration in the **main** container that the developer is interested in.

To create a **development containers** configuration a user would go through the following:

- Analyze the current requirements of the application.
  - Separate requirements to run the application from the ones needed to develop it. (f.e. SDKs, compilation tools, source control, etc.)
- Select a type of [configuration](#configuration-options).
- Define at least the required parameters of each configuration.
- Create a `devcontainer.json` file and set it in the application code.

Additionally the user could think of further configuration options like:

- Environment variables required.
- Tokens and security concerns.
- User permisions.
- Container run configuration. [See `overrideCommand`](devcontainerjson-reference.md#general-devcontainerjson-properties).

In all cases paths are relative to the location of `devcontainer.json`.

A lot of the parameters defined in `devcontainer.json` are for environment specific tools to implement depending on their needs, for example, `portsAttributes`, `remoteUser`, `remoteEnv`, etc.

# Configuration options

## Image based

Image based configurations only reference an image that should be reachable and downloadable through `docker pull` commands. Logins and tokens required for this operations are assumed to be already taken care of. The only required parameter is `image` . The details are [here](devcontainerjson-reference.md#image-or-dockerfile-specific-properties).


## Dockerfile based

This configurations are defined as using a `Dockerfile`  to define the starting point of the **development containers**. As with image based configurations, it is assumed that any base images are already reachable by **docker** when performing a `docker build` command.The only required parameter in this case is the relative reference to the `Dockerfile` in `build.dockerfile`. The details are [here](devcontainerjson-reference.md#image-or-dockerfile-specific-properties).

There are multiple properties that allow users to control `docker build` works:

- `build.context` 
- `build.args`
- `build.target`,
- `build.cacheFrom`

## Docker Compose based

Docker compose configurations utilize `docker-compose`  to create and manage a set of containers required for an application. As with the other configurations, any images required for this operation are assumed to be reachable. The required parameters are:

- `build.dockerComposeFile`  the reference to the docker-compose file(s) to be used.
- `service`  this tag declares the **main** container that will be used for all other operations. Tools are assumed to also use this parameter to connect to the **development container** although they can provide facilities to connect to the other containers as required by the user.

It is important to remark that due to how `docker-compose` works a lot of the properties that apply to **image** and **dockerfile** configurations cant be set by the application and thus must be setup manually by the user using the `docker-compose` format itself.

# Other options

In addition to the configuration options explained above there are other settings that apply when creating the **development containers** to facilitate their use by developers. This properties mostly apply to **image** and **dockerfile** based configurations.

The main ones are:

## Environment Variables

Environment variables can be set in containers at creation time, to override existing ones, or just to be used when a supporting tool connects to the environment.

## Mounts

Mounts allow containers to have access to the underlying machine, share data between containers and to persist information between **development containers**. 

It is important to note that this mounts are from the underlying compute environment and thus in cloud enviroments might not have access to the same data as a local machine.

## Users

Users control the permisions of applications executed in the containers, allowing the developer to control them.

# Lifecycle

A development environment goes through different lifecycle events during their use in the outer and inner loop of development.

- Configuration Validation
- Environment Creation.
- Environment Stop.
- Environment Resume.
- Environment Connection.

## Configuration validation

This section consists of actions required to validate the `devcontainer.json` configuration is valid and does not contain values that conflict with each other.

The tasks executed when validating the configuration are:

- Validate command line parameters are consistent:
  - `workspace-folder` is required since it defines where the `devcontainer.json` file is located.
- Validate access to the `workspace-folder`. Identify that a `devcontainer.json` file is located in the defined locations. [see here](devcontainerjson-reference.md)
- Validate that additional parameters have the correct format.
- Validate that `devcontainer.json`  contains all parameters required for the selected configuration type.

## Environment Creation

### Initialization

During this step the following is executed:
- Validate access to the container orchestrator specified by the configuration.
- Execution of `initializeCommand`.

### Image creation

The first part of an environment creation is generating the final image(s) that the **development containers** are going to use. This step is orchestrator dependent and can consist of just pulling a docker image, running docker build, or docker-compose build. Additionally this step is useful by its own since it permits the creation of intermediate images that can be uploaded and used by other users and thus cut down on creation time. It is encouraged that tools implementing this specification give access to a command that just executes this step.

This step executes the following:

- [Configuration Validation](#configuration-validation) 
- Pull/build/execute of the defined container orchestration format to create images.
- Validate the result of this operations.

### Container creation.

After image creation containers are created based on that image and setup.

This step executes the following:

- Create the container with the specified properties.
- Validate the container(s) were created successfully.

### Post container creation.

- At the end of the container creation step, a set of commands are executed inside the **main** container `on-create-command`, `update-content-command` and `post-create-command`. This set of commands are executed in sequence on a container the first time its created and depending on the creation parameters received, [see here](devcontainerjson-reference.md#lifecycle-scripts) to learn more. By default `post-create-command` is executed in the background after reporting the successful creation of the development environment, this behavior depends on the `wait-for` property defined. 

## Environment stop.

Stops all containers in the environment. The intention of this step is to ensure all containers are stopped correctly to ensure no data is lost.

## Environment Restart

After an environment has been stopped the containers are restarted according to the orchestrator defined. Additionally `post-start-command` is executed in the **main** container.

## Environment Connection

`devcontainer.json`  has `post-attach-command` defined that should be executed after any tools attach to the container. These tools should call for the execution of this command.

# Tool features

Tools implementing this specification (like the reference implementation) are encouraged to support at least the following features:

## Identification

Tools implementing this specificaction should have a way to track what images and containers belong to a particular **development container** allowing the user to keep multiple on an environment and select which one to operate on.

## Definition merge and validation
`read-configuration`

This command should execute all validation tasks on the `devcontainer.json` format as defined [here](devcontainerjson-reference.md).

## Image build
`build`

Executes all steps of the lifecycle up to the build step. This allows users to take this images and publish them.

## Environment creation
`up`

Executes all steps required to obtain a working **development container**, executing tasks in the background as defined [here](#post-container-creation).

## Stop
`stop`

Executes a safe stop of all containers related to the **development container**.

## Stop and Delete
`down`

Executes stop and in addition deletes all containers related to the **development containers**.

## Execute user commands
`run-user-commands`

Runs the asynchronous commands on an environment according to paramenters received. It should be possible for the user to control which ones run. Namely:

- `onCreate`
- `updateContent`
- `postCreateCommand`
- `postAttachCommand`
- `postStartCommand`.

## Execute manual commands
`exec`

Executes commands inside the **main** container of a development environment specified to the user.