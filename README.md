# An environment for our Kubernetes-based ansible projects

![GitHub](https://img.shields.io/github/license/informaticsmatters/kubernetes-ansible-projects)

![GitHub tag (latest SemVer pre-release)](https://img.shields.io/github/v/tag/informaticsmatters/kubernetes-ansible-projects?include_prereleases)

[![Conventional Commits](https://img.shields.io/badge/Conventional%20Commits-1.0.0-yellow.svg)](https://conventionalcommits.org)

A repository with a Visual-Studio (VS) code [devcontainer] containing ansible and
[kubectl]. There is no code here, this repository exists to simplify the development and execution of our kubernetes-based ansible projects by providing a consistent environment
for playbook execution.

Ansible projects are automatically included in this repository using Git
[submodules], which makes each ansible repository a subdirectory of this repository,
but keeps their commits separate.

## Using the repository

---

[ansible]: https://docs.ansible.com
[devcontainer]: https://code.visualstudio.com/docs/devcontainers/containers
[kubectl]: https://kubernetes.io/docs/reference/kubectl/
[submodules]: https://git-scm.com/book/en/v2/Git-Tools-Submodules
