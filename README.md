# Kinetik Tile Development Kit
This repository contains the Tile Development Kit (`TDK`), a set of tools for Tile development. The TDK currently consists of documentation for tile authors and a `Tile Generator` which simplifies the creation of Tiles.

Please review the [Prerequisites](#prerequisites) and then the [Getting Started](#getting-started) section carefully to learn how to use the `Tile Generator` effectively, then study the [Tile Overview](#tile-overview) section to learn more about developing with Tiles.

## Contents
* [Prerequisites](#prerequisites)

* [Getting Started](#getting-started)

    * [Available Tiles](#available-tiles)
    
    * [Tile Development](#tile-development)

        * [Create a Tile](#create-a-tile)

        * [Delete a Tile](#delete-a-tile)

* [Tile Overview](#tile-overview)

    * [Directory Structure](#directory-structure)

    * [Actions](#actions)

    * [Resources](#resources)

    * [Configuration](#configuration)

    * [Automation Runtimes](#automation-runtimes)

    * [Environment](#environment)

* [Tile Use Cases](#tile-use-cases)

* [Contributing](#contributing)

* [License](#license)

---
# Prerequisites

The following prerequisites must be met to use the tools in this repository:

1. [Docker](https://docs.docker.com/get-docker/) installed on your workstation.
2. You must be a member of the [GitHub Kosmos team](https://github.com/orgs/section6nz/teams/kosmos).
3. A GitHub Personal Access Token.
   
    To create a GitHub Personal Access Token, refer to: [GitHub: Creating a Personal Access Token](https://docs.github.com/en/github/authenticating-to-github/creating-a-personal-access-token)

   **NOTE**: The Token must be scoped as follows:
    * `repo`
    * `workflow`
    * `write:packages`
    * `read:org`
    * `delete_repo`

**NOTE**: The TDK, and Tiles more generally, currently leverage GitHub as a matter of practicality. However, no concept,
 principle or architectural decision ties the components to GitHub.

---
# Getting Started

**NOTE**: This section assumes you have met the [prerequisites](#prerequisites).

Before starting a new Tile development effort, it is recommended that you first review the 
[Available Tiles](#available-tiles) to determine if an existing Tile meets your use case, or may be enhanced to do so. 

If no existing Tile satisfies the use case, please refer to [Tile Development](#tile-development) to begin developing a
Tile.

## Available Tiles
To discover the available Tiles, at time of writing, the simplest method is to 
[search for repositories beginning with "kinetik-tile-"](https://github.com/orgs/section6nz/teams/kosmos/repositories?query=kinetik-tile-). In the future, a 'Tile Registry' will be developed to enable effective discovery and distribution of Tiles.

## Tile Development
To begin developing a Tile:

1. Check out this repository on your workstation:

   `git clone git@github.com:section6nz/kinetik-tile-development-kit.git`


2. Set credentials in the environment as follows:

    ```
    export IMAGE_REGISTRY_USERNAME="<GitHub Username>"
    export IMAGE_REGISTRY_TOKEN="<GitHub Personal Access Token>"
    ```
**NOTE**: These credentials (`IMAGE_REGISTRY_USERNAME` and `IMAGE_REGISTRY_TOKEN`) must be set in the
environment to create or delete a Tile.

You are now ready to [Create a Tile](#create-a-tile).

### Create a Tile
Create a tile with the command:

```./tile-generator <TILE_NAME> create```

Optionally, you may also select an automation runtime for your Tile ('ansible' and 'terraform' are supported):

```./tile-generator <TILE_NAME> create terraform```

**NOTE**: The specified runtime will generate default resources from the [repository templates](https://github.com/section6nz/kinetik-repository-templates) repository for Tile development.

Once the Tile has been created successfully, a link will be provided to the Tile repository. You may check out the 
repository and begin developing the Tile. Refer to the `README.md` in the Tile repository for further instructions.

If you are unfamiliar with Tile development, it recommended that you review the [Tile Overview](#tile-overview) section.

### Delete a Tile
**CAUTION:** Ensure the `TILE_NAME` parameter correctly identifies the Tile you wish to delete. Otherwise, you may 
delete a resource that you did not intend.

Delete  a tile with the command:
```./tile-generator <TILE_NAME> delete```

---
# Tile Overview
This section provides an overview of Tiles and the resources therein.

Tiles, in their source code form, are a set of resources in a source code repository. These resources are packaged as an
OCI container image, based on a Kinetik [Automation Runtime](#automation-runtimes) image.

## Directory Structure
The basic directory structure of a Tile can be seen below:

```shell
├── actions
│         ├── configure
│         ├── deploy
│         ├── destroy
│         ├── init
│         ├── undeploy
│         └── verify
├── docs
├── examples
│         └── config.json
├── resources
├── test
│         ├── acceptance
│         └── unit
└── tile.yml
```
## Actions
The Tile actions directory(`/actions`) contains scripts which implement the [Kinetik actions](https://section6.atlassian.net/wiki/spaces/KOS/pages/1501757463/Kinetik+Tiles#Default-lifecycle-actions) and invoke the automation in the
[resources directory](#resources).

The list of tile actions that can be implemented are shown below:
* `configure`
* `deploy`
* `destroy`
* `init`
* `undeploy`
* `verify`

**NOTE**: By default, all the lifecycle actions are not implemented. The tile author will make a conscious choice of the actions required by a Tile to perform its intended functionality, and implement those actions as required.

`kinetik` requires an action to be specified using the `--action` argument when performing operations on a Tile. The
corresponding action script in the Actions directory(`/actions`) is then invoked to perform the action.

The list of actions supported by a Tile is further defined in the [Tile specification](https://section6.atlassian.net/wiki/spaces/KOS/pages/1501757463/Kinetik+Tiles#Tile-Specification) (`.spec.actions[]`).

## Resources
The Resources directory(`/resources`) of the Tile contains the automation implementing the Tile functionality. In the
case of the GitHub tile, the [resources directory](https://github.com/section6nz/kinetik-tile-github/tree/main/resources) contains Ansible automation for automating GitHub functions.

## Configuration
The Tile configuration schema defines the expected configuration for the Tile  and is defined in the Tile specification
using [JSON schema](https://json-schema.org/) as the `spec.config.schema` object. For more information, please refer to
[Tile Specification](https://section6.atlassian.net/wiki/spaces/KOS/pages/1501757463/Kinetik+Tiles#Tile-Specification).

The configuration of a Tile may be shown by describing the Tile (assuming the Tile has been built):

```shell
docker run -it --rm --tile <TILE_NAME> -- describe
# OR
kinetik describe --tile <TILE_NAME>
```

You may generate an example configuration as follows:

```shell
docker run -it --rm --tile <TILE_NAME> -- generate
# OR
kinetik generate --tile <TILE_NAME>
```

## Automation Runtimes
Tiles are built from a Kinetik automation runtime image, all of which include the `kinetik` binary, as well as a
specific automation runtime (e.g. Ansible, Terraform etc). The image from which a Tile is built is defined by the 
Dockerfile by the [FROM directive](https://docs.docker.com/engine/reference/builder/#from).

For further information on automation runtimes, including the specific available runtimes, please refer to the 
[kinetik-runtimes repository](https://github.com/section6nz/kinetik-runtimes).

## Environment
The Tile path is set inside the Kinetik base image by the environment variable `KINETIK_TILE_PATH`:

```shell
$ docker run --rm --entrypoint env \
    docker.pkg.github.com/section6nz/kinetik-tile-github/kinetik-tile-github -- \
      | grep '^KINETIK_TILE_PATH'
KINETIK_TILE_PATH=/kinetik/tile/
```
`KINETIK_TILE_PATH` obviates the need to pass the `--tile-path <PATH>` argument to the Tile when running the Tile.

---
# Kinetik
Kinetik is a self-contained binary (`kinetik`) which implements the Tile interface. `kinetik` is packaged and 
distributed as a container image called [kinetik-base](https://github.com/section6nz/kinetik-base).

**NOTE**: At time of writing, the `kinetik` binary is not published to a central repository and must be built from
source.

`kinetik` is included by default inside every Tile and performs the function of Tile entrypoint. For example, when the
GitHub Tile is run, `kinetik` is invoked inside the GitHub Tile:

```
$ docker run docker.pkg.github.com/section6nz/kinetik-tile-github/kinetik-tile-github

Kinetik
Version 0.8.0
SECTION6

"Kinetik is the Kosmos CLI."

USAGE:
    kinetik [FLAGS] <SUBCOMMAND>

FLAGS:
    -h, --help       Prints help information
    -V, --version    Prints version information
    -v, --verbose    Verbose output

SUBCOMMANDS:
    describe    Describe a resource
    generate    Generate configuration
    help        Prints this message or the help of the given subcommand(s)
    run         Run a resource
    validate    Validate configuration
```

**NOTE**: It is not necessary to build and/or obtain `kinetik` in order to create and delete Tiles. This section is included for informational purposes.

---
# Tile Use Cases
Some practical use cases for tiles will be collated in another document as tiles are created and used in production.
The use case example documentation is intended for tile users to show how current available Tiles can be used to create intended products.

Some preliminary examples of how tiles can be used are:

| Type      | Examples|
| :---        |    :----:   | 
| CI/CD Integration      | Integrating [ArgoCD](https://argoproj.github.io/argo-cd/), [Jenkins](https://www.jenkins.io/)|
| Deployment  | Generating different types of [Kubernetes static deployment](https://kubernetes.io/docs/concepts/workloads/controllers/) files |
| Git repository management | Integrating [GitHub](https://github.com/), [GitLab](https://about.gitlab.com/), [BitBucket](https://bitbucket.org/)|

## Tile Example Demo

This is an example for demoing how a Tile can be created and developed to execute tasks for your project. An example
demo is also provided, please refer to
[example demo video](https://drive.google.com/file/d/1t4HykCJA0n0m9PSvz0vAxaV3AHnKh2aF/view?usp=sharing).

The example includes two parts: Tile creation and Tile development:

### Tile creation
1. Checkout the Github repository on the workstation:
```
git clone https://github.com/section6nz/kinetik-tile-development-kit.git
```

2. Set up the credentials:
```
export IMAGE_REGISTRY_USERNAME="<GitHub Username>"
export IMAGE_REGISTRY_TOKEN="<GitHub Personal Access Token>"
```

3. Create a Tile named ```kosmos-demo```, and choose Ansible as the automation runtime:
```
cd kinetik_tile_development_kit
./tile-generator kosmos-demo create ansible
```

4. When the creating is finished, the link of the Tile will be shown at last:
```
Tile kosmos-demo created >> https://github.com/section6nz/kinetik-tile-kosmos-demo <<
```
Open this link, the newly created repository can be accessed. 

The whole repository is automatically generated from the tile_generator. The 
README file is provided for details about the new Tile.

### Tile development

Checkout the kosmos-demo Tile to your workstation, adding the automation tasks for the Tile to implement for your project.

1. Checkout the kosmos-demo Tile on the workstation and review all the resources created automatically:
```
git clone https://github.com/section6nz/kinetik-tile-kosmos-demo
```

2. Build the project
```
cd kinetik-tile-kosmos-demo
make build
```

3. Run the Tile container image using ```docker``` command
```
docker run -it --rm \
  --volume ${PWD}/examples/config.json:/tile/config.json \
  docker.pkg.github.com/section6nz/kinetik-tile-kosmos-demo/kinetik-tile-kosmos-demo \
    --config file:///tile/config.json \
    --action init
```
From the command, the ```init``` action was invoked, the tasks of init action is defined in the Ansible playbook
in the resources directory (```resources/roles/mytile/tasks/init.yml```).
A "Hello World" message should be seen after running the command.

4. Define your own tasks:

The ```resources``` directory contains the automation tasks for your project, All the automation tasks you want the 
tile implements should be defined in the action YAML files.
By default, all the actions just print a "Hello World" message. 

1. set up your tasks in the action file

For example, in the following ```init.yml``` file, a new task can be added for Tile to create a new Github repository for holding the project:
```
# init.yml

- name: init tile
  ansible.builtin.debug:
    msg: 'Hello World from the init action'

- name: Login to Github with token
  shell: echo {{ repositoryToken }} | gh auth login --with-token

- name: Create the repository
  shell: |
    gh repo create {{ repositoryOwner }}/{{ repositoryName }} --description "{{ repositoryDescription }}" --private --confirm;
    git remote set-url origin git@github.com:{{ repositoryOwner }}/{{ repositoryName }}.git;
    
```

2. Clarify the schema of Tile

The variables included in the double curly brackets in the ```init.yml``` are the required to connect and authenticate for your Github account, all the information should
be clarified in the schema of the Tile config(```/tile/yml``):
```
# tile.yml

schema: |
      {
       	"$schema": "http://json-schema.org/draft-07/schema#",
      	"$id": "https://section6.nz/kinetik/tile/schemas/kinetik-tile-demo-configuration.json",
      	"title": "tile-schema-configuration-kinetik-tile-demo",
      	"description": "kinetik-tile-demo configuration schema",
      	"type": "object",
        "properties": {
                "repositoryUsername": {
                  "description": "The username used to manage resources",
                  "type": "string"
                },
                "repositoryToken": {
                  "description": "The access token used to manage resources",
                  "type": "string"
                },
                "repositoryName": {
                  "description": "The repository name",
                  "type": "string"
                },
                "repositoryDescription": {
                  "description": "The repository description",
                  "type": "string"
                },
                "repositoryOwner": {
                  "description": "The repository owner",
                  "type": "string"
                },
                "repositoryTemplateType": {
                  "description": "The repository template type",
                  "type": "string"
                }
        },
        "required": [
                    "repositoryUsername",
                    "repositoryToken",
                    "repositoryName",
                    "repositoryOwner",
                    "repositoryDescription",
                    "repositoryTemplateType"
        ]
```
The values of these required variable is stored in the config file (```/config.json```):
```
{
  "repositoryUsername": "<USERNAME>",
  "repositoryToken": "<PERSON_ACCESS_TOKEN>",
  "repositoryName": "test-demo",
  "repositoryDescription": "This is test",
  "repositoryOwner": "NAME",
  "repositoryTemplateType": "kinetik-tile-ansible",
}
```

3. Install requirements

To run the task defined in the init action, Github CLI(```gh```) is used. Before running the actions, ```gh``` should be installed.
In the ```DOCKERFILE```, all the installation is clarified:
```
# DOCKERFILE

# Install requirements.
ARG GITHUB_CLI_VERSION=1.6.2

RUN apk add git --update --no-cache \
    && wget -qO- \
        https://github.com/cli/cli/releases/download/v${GITHUB_CLI_VERSION}/gh_${GITHUB_CLI_VERSION}_linux_amd64.tar.gz \
        | tar xfz - -C /tmp/ \
    && mv /tmp/gh_${GITHUB_CLI_VERSION}_linux_amd64/bin/gh /usr/bin \
    && chown root:root /usr/bin/gh \
    && chmod 755 /usr/bin/gh \
    && rm -rf /tmp/gh_${GITHUB_CLI_VERSION}_linux_amd64

```

Then build the Tile firstly and run the same docker command in step 3, the "Hello World from the init action will be displayed", at the same time,
a new Github repository called "test-demo" is created under your account.


---
# Contributing

Please refer to [CONTRIBUTING.md](CONTRIBUTING.md).

---
# License

Please refer to [LICENSE.md](LICENSE.md).
