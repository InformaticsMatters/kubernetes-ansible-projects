# An environment for our Kubernetes-based ansible projects

![GitHub tag (latest SemVer pre-release)](https://img.shields.io/github/v/tag/informaticsmatters/kubernetes-ansible-projects?include_prereleases)

[![Conventional Commits](https://img.shields.io/badge/Conventional%20Commits-1.0.0-yellow.svg)](https://conventionalcommits.org)

A repository with a Visual-Studio (VS) code [devcontainer] containing ansible and
[kubectl]. There is no code here, this repository exists to simplify the development and execution of our kubernetes-based ansible projects by providing a consistent environment
for playbook execution.

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

- Docker
- A `${HOME}/k8s-config` directory
- A `${HOME}/.kube` directory

## Using the repository (VS Code)

When you open this repository in VS-Code after cloning you will be told the folder
contains a _Dev Container configuration file_ and will be invited to reopen the folder
in a container. Do so or select the command *Reopen in Container* using the
**Command Palette** (`Ctrl+Shift+P` or `Cmd+Shift+P` on MacOS).

If the container does not exist VS Code will take a few minutes to build it before
presenting you with a terminal in it where you will find a dedicated `ansible`
and `kubectl`.

If this is a fresh clone of the repository you will need to populate the submodule
directories before you can use them - Git does not do this automatically without help.
To do this after cloning just run the following command in the terminal: -

    git submodule update --init --recursive

>   When you clone this repository you can add the command-line option
    `--recurse-submodules` to do this automatically

>   To read more about how submodules work read Git's [submodules] documentation

Now that you have a fully populated project (and Ansible) you can now run playbooks
by moving to the relevant submodule directory.

To run our basic **ansible-infrastructre** playbook using parameters for a local
cluster simply change to its directory and run the `site.yaml` file: -

    pushd ansible-infrastructure
    ansible-playbook site.yaml -e @parameters-local.yaml

>   Each ansible project is responsible for its own inventory, and configuration.

---

[ansible]: https://docs.ansible.com
[devcontainer]: https://code.visualstudio.com/docs/devcontainers/containers
[docker]: https://www.docker.com
[kubectl]: https://kubernetes.io/docs/reference/kubectl/
[submodules]: https://git-scm.com/book/en/v2/Git-Tools-Submodules
