This repository contains **no application or playbook code of its own**. It is a
convenience wrapper that:

1. Aggregates several independent kubernetes-based Ansible projects as Git
   **submodules**, each living in its own top-level directory.
2. Provides a VS Code **devcontainer** (and an equivalent virtualenv recipe) that
   ships `ansible`, `ansible-lint`, and `kubectl` so every submodule runs in a
   consistent environment.

Each submodule is a self-contained Ansible project with its own inventory,
parameters, and lifecycle. You can use any of them without this wrapper; this
repo just makes it convenient to work with all of them at once.

## Submodules

Defined in `.gitmodules`. Note that the default branch differs per submodule
(this matters because submodules clone in a detached HEAD state — check out the
branch below before committing inside one):

| Directory | Host | Branch |
|---|---|---|
| `ansible-infrastructure` | GitHub | `main` |
| `squonk2-account-server-ansible` | GitLab (private) | `master` |
| `squonk2-data-manager-ansible` | GitLab (private) | `master` |
| `squonk2-data-manager-job-operator-ansible` | GitHub | `main` |
| `squonk2-data-manager-jupyter-operator-ansible` | GitHub | `main` |
| `squonk2-data-manager-ui-ansible` | GitHub | `master` |
| `squonk2-data-manager-viz-operator-ansible` | GitHub | `main` |
| `squonk2-fastapi-ws-event-stream-ansible` | GitHub | `main` |

Two submodules are hosted on **private GitLab** repos — cloning/updating fails
without GitLab access.

## Commands

### Populating / updating submodules

```bash
# Fresh clone with submodules:
git clone https://github.com/InformaticsMatters/kubernetes-ansible-projects --recurse-submodules

# If already cloned without submodules:
git submodule update --init --recursive

# Pull everything (root + all submodules) from their remotes:
git pull --recurse-submodules
```

If submodule directories are empty, they have not been initialised — run the
`git submodule update --init --recursive` command above before doing anything
else.

### Non-devcontainer environment (virtualenv)

```bash
python -m venv venv
source venv/bin/activate
pip install --upgrade pip
pip install -r .devcontainer/requirements.txt
export KUBECONFIG=~/.kube/config   # playbooks do NOT assume a default kubeconfig
```

### Running a playbook

Each submodule owns its inventory and config, so you must `cd` into it first:

```bash
pushd ansible-infrastructure
ansible-playbook site.yaml -e @parameters-local.yaml
```

The `-e @parameters-<env>.yaml` pattern (extra-vars from a parameters file) is
the convention for selecting a target environment.

## Key conventions and constraints

- **Pinned dependencies**: `.devcontainer/requirements.txt` requires every entry
  to use `==` (currently ansible 12.0.0, ansible-lint 25.9.1, kubernetes 32.0.1,
  pre-commit 4.3.0). `kubectl` is pinned in `.devcontainer/Dockerfile`
  (currently v1.32.8) and the clusters use kubectl v1.32.
- **kubeconfig**: the devcontainer mounts `~/.kube` and `~/k8s-config` and sets
  `KUBECONFIG=/home/vscode/.kube/config`. Outside the devcontainer you must set
  `KUBECONFIG` yourself.
- **Local clusters need a hostname, not 127.0.0.1**: from inside the container,
  `127.0.0.1` will not reach the host's cluster. Add an alias such as
  `kubernetes.docker.internal` to the host `/etc/hosts` and point the kubeconfig
  `server:` at it (see README for details).
- **Image tags / pull secrets** are injected as `IM_DEV_*` environment variables
  (see `.devcontainer/devcontainer.json` `containerEnv`); they pass through from
  the host environment.
- **Secrets**: `**/*.password` files are git-ignored — never commit them.
- **Commits** follow Conventional Commits.

## Editing submodule code

Changes inside a submodule are tracked by that submodule's own repo, not this
one. Before committing inside a submodule, `git checkout` its correct branch
(see the table above) — submodules are checked out detached by default. Consult
the Git submodules documentation's "Working on a Submodule" section before
publishing changes.
