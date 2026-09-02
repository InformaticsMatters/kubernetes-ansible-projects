---
name: install-squonk2-component
description: Install or upgrade a Squonk2 component (Account Server, Event Stream, Data Manager, UI, or the Job/Jupyter/Viz operators) into a named Squonk2 installation by running the component's Ansible playbook. Use when asked to install, deploy, upgrade, or bump the version of a Squonk2 component in an installation such as local, dls-dev, dls-test, dls-prod, or scw-production.
---

# Install a Squonk2 component

Runs one component's `site.yaml` against a named Squonk2 _installation_.
Background and conventions live in `INSTALLATION-GUIDE.md` at the repository root.

Each component is an Ansible project held here as a Git submodule. Installing a
component means: pick the component, pick the installation, supply the vault
password and `KUBECONFIG`, choose the image tag, then run the playbook.

## Components

| Component | Submodule directory | Var prefix | Extra playbook |
|---|---|---|---|
| Account Server | `squonk2-account-server-ansible` | `as_` | — |
| Event Stream (ESS) | `squonk2-fastapi-ws-event-stream-ansible` | `ess_` | — |
| Data Manager | `squonk2-data-manager-ansible` | `dt_` | — |
| Data Manager UI | `squonk2-data-manager-ui-ansible` | `ui_` | — |
| Job Operator | `squonk2-data-manager-job-operator-ansible` | `jo_` | — |
| Jupyter Operator | `squonk2-data-manager-jupyter-operator-ansible` | `jo_` | `site_dm.yaml` |
| Viz Operator | `squonk2-data-manager-viz-operator-ansible` | `svo_` | `site_dm.yaml` |

The two variables that matter are `<prefix>_installation_name` and `<prefix>_image_tag`.

> The Data Manager's prefix is `dt_` ("data tier"), **not** `dm_`, despite the
> submodule and namespace both being named "data-manager".

The Event Stream installs into the Account Server's namespace; every other
component creates its own. For a fresh installation the logical order is
Account Server -> Event Stream -> Data Manager -> UI -> Job/Jupyter/Viz operators.

## Procedure

Follow these steps in order. Stop and ask the user whenever a check fails —
never guess an installation name, a version, or a cluster.

### 1. Establish the component and installation

Take both from the user's request. Ask only for what is missing. Resolve the
component to a row in the table above and `cd` nowhere yet — all path checks
below are relative to the repository root.

### 2. Check the vault password file

The vault password is expected at the repository root, named for the
installation:

```bash
ls -l <installation>-installation.password
```

If it is absent, **stop**. Tell the user the file is missing and that the keys
are held in KeePass under **Squonk -> Installations**. Do not fall back to
`--ask-vault-pass` unless the user asks for it.

If it is present, tighten its permissions (Ansible warns otherwise):

```bash
chmod 600 <installation>-installation.password
```

`**/*.password` is git-ignored, so these files are never committed.

### 3. Check the component has a vault for this installation

```bash
ls <submodule>/roles/*/vars/sensitive-<installation>.vault
```

Most components ship vaults only for the installations they have been deployed
to. If there is no matching vault, stop and report which installations that
component does have — the user has probably named the wrong installation.

### 4. Show the KUBECONFIG and confirm the cluster

Display the value plainly, along with the context the playbook will actually
act on, and let the user confirm before anything is applied:

```bash
echo "KUBECONFIG=${KUBECONFIG:-<unset>}"
kubectl config current-context
kubectl config view --minify -o jsonpath='{.clusters[0].cluster.server}'; echo
```

If `KUBECONFIG` is unset, **stop** and ask the user which kubeconfig to use —
the playbooks do not assume a default. Report the context and server back to
the user and get their agreement that it is the right cluster for this
installation before continuing.

### 5. Ask for the version

Ask the user which version (image tag) of the component to install. Do not
assume, and do not reuse a tag from an earlier run in the conversation without
confirming it. Tags are semver `X.Y.Z` with no `v` prefix, though `latest` and
`trunk` are also valid for development installations.

It is often useful to show what is deployed now, so the user can see what they
are moving from:

```bash
kubectl get deployment -n <namespace> -o jsonpath='{.items[*].spec.template.spec.containers[*].image}'; echo
```

### 6. Run the playbook

Run from inside the submodule, using this repository's virtualenv:

```bash
cd <submodule>
ANSIBLE_VAULT_PASSWORD_FILE=../<installation>-installation.password \
ANSIBLE_COLLECTIONS_PATH=../.venv/lib/python3.13/site-packages \
../.venv/bin/ansible-playbook site.yaml \
  -e <prefix>_installation_name=<installation> \
  -e <prefix>_image_tag=<version>
```

Add `-e @parameters-<installation>.yaml` **only if that file exists** in the
submodule — several installations have no parameters file and passing a missing
one fails the run.

`ANSIBLE_COLLECTIONS_PATH` is set deliberately: a stale `kubernetes.core` in
`~/.ansible/collections` otherwise shadows the version shipped with the pinned
Ansible. Adjust the `python3.13` path element if `.python-version` moves.

If `.venv` does not exist, set it up first:

```bash
uv venv && uv pip install --python .venv/bin/python -r .devcontainer/requirements.txt
```

### 7. Run the DM-side playbook where applicable

The Jupyter and Viz operators install additional objects (typically
ConfigMaps) into the Data Manager namespace. After `site.yaml` succeeds, run
`site_dm.yaml` from the same directory with the same variables and environment.

### 8. Report

Say plainly whether the run succeeded, and quote Ansible's `failed=` counts from
the play recap. If tasks failed, show the failing task output rather than
summarising it away. The playbooks are idempotent, so a re-run after a fix is
safe.

## Notes

- Never commit a `.password` file, and never print a vault password or decrypted
  vault content into the conversation.
- These playbooks change a live cluster. Everything up to step 5 is read-only;
  step 6 is the point of no return, so make sure the user has confirmed the
  cluster and the version before running it.
- To edit sensitive values rather than install, use `ansible-vault edit` on the
  relevant `sensitive-<installation>.vault` with the same password file.
