# An environment for our ansible projects

[![Conventional Commits](https://img.shields.io/badge/Conventional%20Commits-1.0.0-yellow.svg)](https://conventionalcommits.org)

A repository with a Visual-Studio (VS) code [devcontainer] with [ansible] and
[kubectl] built-in. There is no code here, this repository exists to simplify the
development and execution of our kubernetes-based ansible projects by providing a
consistent environment for playbook execution.

Ansible projects are automatically included in this repository using Git
[submodules], which makes each ansible repository a subdirectory of this repository,
but keeps their commits separate.

Every submodule is an isolated kubernetes-based ansible project. You do not
have to use this project to use them - it's simply a convenient way to use all of them.
The devcontainer provides you with all the tools you need while also mapping our
kubernetes config files into the `${HOME}` directory of the container. `KUBECONFIG`
is also set automatically to `${HOME}/.kube/config` so any kubernetes config file
you have there will be available straight away.

Prerequisites: -

1.  Docker
2.  A `${HOME}/k8s-config` directory
3.  A `${HOME}/.kube` directory

As the project uses submodules, if you've not used them before, it might be worth
spending some time reading about how submodules behave -especially if you plan on
editing the sub-module's code.

## Initialising the clone

As we use submodules, the easiest way to clone this repository is with the following
command: -

    git clone https://github.com/InformaticsMatters/kubernetes-ansible-projects --recurse-submodules

If you have already cloned the repository without initialising the submodules
you will need to populate the submodule directories before you can use them.
To do this after cloning just run the following command from inside the repository: -

    git submodule update --init --recursive

To read more about how submodules work read Git's [submodules] documentation

## Preparation (VS Code)

After cloning, when you open this repository in VS-Code you will be told the folder
contains a _Dev Container configuration file_ and will be invited to reopen the folder
in a container. Do so or select the command *Reopen in Container* using the
**Command Palette** (`Ctrl+Shift+P` or `Cmd+Shift+P` on MacOS).

If the container does not exist VS Code will take a few minutes to build it before
presenting you with a terminal in it where you will find the tools `ansible`
and `kubectl`.

## Preparation (virtual environment)

It's not then end of the world if you can't use VS Code and its devcontainer.

If you are using a _recent_ Python you can use a its built-in virtual environment
capability to create an execution environment containing Ansible and some other tools.

It should be simple to get started by running these commands from the root
of the clone: -

    python -m venv venv
    source venv/bin/activate
    pip install --upgrade pip
    pip install -r .devcontainer/requirements.txt

>   As along as the requirements can be installed then any Python should be suitable
    but we recommend recent versions, and currently use 3.12 and 3.13.

If you already use our kubernetes clusters you probably have `kubectl`
installed. If not you now need to [install kubectl] on your host,
one with a version that's suitable for our clusters. We currently use kubectl v1.32.

You will also need to set a suitable value for the `KUBECONFIG` environment
variable. Unlike `kubectl`, the playbooks do not _assume_ that you want to use
`~/.kube/config`. If you want to use the default config file you need to set the
variable accordingly, like this: -

    export KUBECONFIG=~/.kube/config

## Running the playbooks

With your environment set (VS code or a virtual environment) you should now
be able to run our playbooks by moving to the relevant submodule directory.

To run our basic **ansible-infrastructure** playbook (using our built-in parameters
for a local cluster) simply change to its directory and run the `site.yaml` file: -

    pushd ansible-infrastructure
    ansible-playbook site.yaml -e @parameters-local.yaml

>   Each ansible project is responsible for its own inventory, and configuration.
    This is one reason why you need to move to the corresponding repository
    sub-directory before running a playbook.

## Updating the clone

To update the root project you can use a standard `git pull` but this will not
update the content of your submodules. Read the [submodule] documentation
**Pulling Upstream Changes from the Project Remote** section for a better understanding
of how the man project and its submodules behave.

To pull everything from the remote you can run this command: -

    git pull --recurse-submodules

## Working on a submodule

Editing submodule code correctly is often a complicated affair.
You have to do some extra steps if you want changes in a submodule to be tracked.
As a result you are _strongly advised_ to first read the [submodule] documentation's
**Working on a Submodule** section for a better understanding of how to edit and
commit any changes _before_ you make any changes to the submodule code.

---

[ansible]: https://docs.ansible.com
[devcontainer]: https://code.visualstudio.com/docs/devcontainers/containers
[docker]: https://www.docker.com
[install kubectl]: https://kubernetes.io/docs/tasks/tools/#kubectl
[kubectl]: https://kubernetes.io/docs/reference/kubectl/
[submodule]: https://git-scm.com/book/en/v2/Git-Tools-Submodules
[submodules]: https://git-scm.com/book/en/v2/Git-Tools-Submodules
