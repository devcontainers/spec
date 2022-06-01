# Dev container specification

The purpose of the **development container** specification is to provide a way to enrich containers with the content and metadata necessary to enable development inside them. These container **environments** should be easy to use, create, and recreate. 

A **development container** is a container in which a user can develop an application.  Tools that want to implement this specification should provide a set of features/commands that give more flexibility to users and allow **development containers** to scale to large development groups.

An **environment** is defined as a logical instance of one or more **development containers**, along with any needed side-car containers. An environment is based on one set of metadata that can be managed as a single unit. Users can create multiple **environments** from the same configuration metadata for different purposes.

Historically, dev containers have been used to enable a development workflow used either locally or in the cloud, and they've been focused on using **Docker** or **Docker Compose**. The intent of this specification and related tools is to expand the reach of **development containers**, allow the usage of containers by themselves or different orchestration technologies, and allow any tool to manage and create them.

The focus of the dev container specification is to describe how to enrich a container for the purposes of development, rather than acting as a multi-container orchestrator format.  Container orchestrator formats can be referenced, when needed, to manage multiple containers and the **development container** lifecycle. Today, `devcontainer.json` includes scenario-specific properties for working without a container orchestrator (by directly referencing an image or Dockerfile) and for using Docker Compose as a simple multi-container orchestrator.

# Metadata

**Development containers** allow one to define a repeatable development environment for a user or team of developers that includes the execution environment the application needs. A development container defines an environment in which you develop your application before you are ready to deploy. While deployment and development containers may resemble one another, you may not want to include tools in a deployment image that you use during development.

A **development container** is composed of a definition (e.g. contained in a `devcontainer.json` file) that deterministically creates containers under the control of the user.

## `devcontainer.json`

The current metadata schema for **development containers** is contained in a `devcontainer.json` file.

The `devcontainer.json` specification contains different configuration options related to the underlying orchestrator format. At the same time, this specification leaves space for further development and implementation of other orchestrator mechanisms and file formats.

While this metadata may be provided in different ways, when searching for a devcontainer.json file, products should expect to find a devcontainer.json file in one or more of the following locations (in order of precedence):

- .devcontainer/devcontainer.json
- .devcontainer.json
- .devcontainer/**/devcontainer.json (where ** is a sub-folder)

It is valid that these files may exist in more than one location, so consider providing a mechanism for users to select one when appropriate.


# Configuration options

**Development containers** currently support multiple ways to specify the containers that create an environment. The following section describes the differences between them.

## Image based

Image based configurations only reference an image that should be reachable and downloadable through `docker pull` commands. Logins and tokens required for these operations are execution environment specific. The only required parameter is `image`. The details are [here](devcontainerjson-reference.md#image-or-dockerfile-specific-properties).

## Dockerfile based

These configurations are defined as using a `Dockerfile` to define the starting point of the **development containers**. As with image based configurations, it is assumed that any base images are already reachable by **Docker** when performing a `docker build` command. The only required parameter in this case is the relative reference to the `Dockerfile` in `build.dockerfile`. The details are [here](devcontainerjson-reference.md#image-or-dockerfile-specific-properties).

There are multiple properties that allow users to control how `docker build` works:

- `build.context` 
- `build.args`
- `build.target`,
- `build.cacheFrom`

## Docker Compose based

Docker Compose configurations use `docker-compose` to create and manage a set of containers required for an application. As with the other configurations, any images required for this operation are assumed to be reachable. The required parameters are:

- `build.dockerComposeFile`: the reference to the Docker Compose file(s) to be used.
- `service`: declares the **main** container that will be used for all other operations. Tools are assumed to also use this parameter to connect to the **development container**, although they can provide facilities to connect to the other containers as required by the user.

It is important to remark that due to how `docker-compose` works a lot of the properties that apply to **image** and **dockerfile** configurations can’t be set by the application and thus must be set up manually by the user using the `docker-compose` format itself.

-`runServices`: the set of services in the `docker-compose` configuration that should be started or stopped with the environment.

# Other options

In addition to the configuration options explained above, there are other settings that apply when creating **development containers** to facilitate their use by developers. These properties mostly apply to **image** and **Dockerfile** based configurations.

We describe the main ones below.

## Environment Variables

Environment variables can be set in containers at creation time, to override existing ones, or just to be used when a supporting tool connects to the environment.

There are two kinds of environment variables:
* Container: These variables are part of the container when it is created and are available at all points in its lifecycle.
* Remote: This variables are to be set by a **development container** supporting tool as part of its environment when it runs. These variables can change during the lifetime of the container.

## Mounts

Mounts allow containers to have access to the underlying machine, share data between containers and to persist information between **development containers**. 

It is important to note that these mounts are from the underlying compute environment and thus cloud environments might not have access to the same data as a local machine.

## Workspace folder

The `workspace-folder` is used for the purpose of identifying the path where the configuration files are found. This path is also automatically included in the mounted folders for the container. This mount is by default created pointing to `/workspace` but can be modified with the [`workspaceMount` and `workspaceFolder`](devcontainerjson-reference.md#image-or-dockerfile-specific-properties) properties.

> [!NOTE]
> Its important that this is not considered in the case of [Docker compose](#docker-compose-based).

## Users

Users control the permissions of applications executed in the containers, allowing the developer to control them. The specification takes into account two types of user definitions:

* Container User: The user that will be used for all operations that run inside a container. This allows the ENTRYPOINT for the image to execute with different permissions than the developer.
* Remote User: Is used to run the [lifecycle](#lifecycle) scripts insde the container. Also this is the user that tools and editors that connect to the container should use to run their processes.

# Lifecycle

A development environment goes through different lifecycle events during its use in the outer and inner loop of development.

- Configuration Validation.
- Environment Creation.
- Environment Stop.
- Environment Resume.
- Environment Connection.

## Configuration Validation

This section consists of actions required to validate the `devcontainer.json` configuration and does not contain values that conflict with each other.

The tasks executed when validating the configuration are:

- `workspace-folder` is required since it defines where the `devcontainer.json` file is located and identifies the configuration to be used for the environment.
- Validate access to the `workspace-folder`. 
- Validate that the metadata (for example `devcontainer.json`) contains all parameters required for the selected configuration type.

## Environment Creation

The creation process goes through the steps necesarry to go from the user configuration to a working **environment** that is ready to be used.

### Initialization

During this step, the following is executed:
- Validate access to the container orchestrator specified by the configuration.
- Execution of `initializeCommand`.

### Image creation

The first part of environment creation is generating the final image(s) that the **development containers** are going to use. This step is orchestrator dependent and can consist of just pulling a Docker image, running Docker build, or docker-compose build. Additionally, this step is useful on its own since it permits the creation of intermediate images that can be uploaded and used by other users, thus cutting down on creation time. It is encouraged that tools implementing this specification give access to a command that just executes this step.

This step executes the following:

- [Configuration Validation](#configuration-validation) 
- Pull/build/execute of the defined container orchestration format to create images.
- Validate the result of these operations.

### Container Creation

After image creation, containers are created based on that image and setup.

This step executes the following:

- Create the container with the specified properties.
- Validate the container(s) were created successfully.

### Post Container Creation

- At the end of the container creation step, a set of commands are executed inside the **main** container: `onCreateCommand`, `updateContentCommand` and `postCreateCommand`. This set of commands is executed in sequence on a container the first time it's created and depending on the creation parameters received. You can learn more in the [documentation on lifecycle scripts](devcontainerjson-reference.md#lifecycle-scripts). By default, `postCreateCommand` is executed in the background after reporting the successful creation of the development environment.
- If the `waitFor` property is defined, then execution should stop at the specified property.
- `userEnvProbe` is used to define the way environment variables are read from the container before executing the lifecycle hooks.

## Environment Stop

Stops all containers in the environment. The intention of this step is to ensure all containers are stopped correctly to ensure no data is lost.

## Environment Restart

After an environment has been stopped, the containers are restarted according to the orchestrator defined. Additionally, `postStartCommand` is executed in the **main** container.
