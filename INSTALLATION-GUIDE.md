# The Squonk2 Installation Guide
Squonk2 consists of many components. They are all compiled into
container images (typically built for AMD and ARM CPU architectures)
and installed into Kubernetes using Ansible **Playbook Roles**, where we create a
**Namespace** for each component.

Behind many squonk2 components is the _infrastructure_. A **Namespace** that consists
of a set of centralised services considered "common" to two or more components.
The non-negotiable part of the infrastructure is a RabbitMQ message broker,
usually installed using the **RabbitmqCluster** operator.

We often find a Keycloak service for authentication and authorisation, but this may be
located elsewhere, and although it must exist, is not considered a mandatory part of the
_infrastructure_.

A key assumption when installing Squonk2 is that the infrastructure exists,
and Ansible variables in each components's Role are used to describe it.

## Background
This repository is an 'umbrella' repository, a convenient main repository
where the Ansible repositories for all the Squonk2 installations can be found as
Git _sub-modules_. checking out this repository should enable you to see
all the playbook repositories.

Here we discus the playbooks, sensitive files and their encryption keys
before explaining the installation process.

### Playbooks
Components often require a significant number of Kubernetes objects in order to
function. The objects are all defined by Ansible playbook `templates` (with each
object in its own template file) with the component's **Role** directory.

Each component's playbook consists of at least one play - the `site.yaml` file.

All the variables in a component's playbook have a component-specific prefix.
Although it does not follow any Ansible convention it does allow us to create
one variable file that can be used for more than one component.
Although we rarely do this - it is a design feature and a useful pattern that
we should continue to follow.

The variable prefixes for each component are: -

- `as_` (Account Server)
- `dt_` (Data Manager/Tier)
- `ui_` (UI)
- `ess_` (Event Stream Service)
- `jo_` (Job/Jupyter Operator)
- `svo_` (Viz Operator)

### Encrypted sensitive configuration
We store sensitive information (in Ansible variables) using Ansible Vault,
and we put all the information (for each component) in a file
called `sensitive-[installation name].vault` in the role's `vars` directory.

Not all components have or need sensitive information but those that do
use this pattern.

### Installations and encryption keys
We give each Squonk2 _installation_ a `name` (an RFC 1123 compliant string).
The name is primarily used to identify the sensitive files that apply to that
installation. For example, the `scw-production` installation's sensitive files
will be encrypted into file called `sensitive-scw-production.vault`.

The encryption password for a vault file is the same for all the vault files
for the same installation. So the Account Server, Data Manager, and UI for example
will all share the same encryption key. Ideally, the key is also different for
each installation but it doesn't have to be.

Using the vaults in this way we can commit sensitive material to GitHub (or GitLab)
and distribute the encryption keys only tho those who need them, while keeping
the sensitive material for the remaining installations secret.

To ensure that Ansible can decrypt the sensitive file, we place the `name` of
installation in a `[component]_installation_name` playbook variable (often on the command-line). The Account Server's variable is `as_installation_name` and it uses
this value to load the corresponding sensitive file.

We then use any one of the standard Ansible patterns to provide the
encryption key to the running playbook: -

- Using `--ask-vault-pass` when running the playbook to enter the password
  interactively
- A password file using either the `--vault-password-file`
  or `ANSIBLE_VAULT_PASSWORD_FILE` environment variable

In this example for the Account Server, we've put the encryption key for the
`scw-scaleway` installation into `~/scw-scaleway-installation.password` and then
choose the version of the component (`4.7.2` here): -

```bash
pushd squonk2-account-server-ansible
chmod 600 ~/scw-scaleway-installation.password
export ANSIBLE_VAULT_PASSWORD_FILE=~/scw-scaleway-installation.password
export KUBECONFIG=~/my-kubeconfig
ansible-playbook site.yaml -e as_installation_name=scw-scaleway -e as_image_tag=4.7.2
```

We keep our encryption keys in **KeyPass** in the **Squonk -> Installations** group.
There (at the time of writing) you will find keys for installations called `local`,
`dls-dev`, `dls-prod`, `dls-test`, and `scw-production`.

> The `local` is used for local kubernetes clusters (e.g. **Docker** or **kubeadm** ).

## Installation
Now that we've discussed the playbooks, installation names, and vault keys
all that remains is the Kubernetes config file (typically providing admin privileges).

Armed with an installation `name`, its encryption `key`, and the `KUBECONFIG` for the
Kubernetes cluster the installation belongs to, we can install a Squonk2.

There is a _logical_ order for a _fresh_ installation. Again, assuming the
_infrastructure_ the installation relies on exists the order is: -

1. Account Server (AS)
2. The FastAPI WebSocket Event Stream (which lives in the AS Namespace)
3. The Data Manager (DM)
4. The Data Manager UI (UI)
5. The various operators: -
   1. The Job Operator
   2. The Jupyter Operator
   3. The Viz Operator

Apart from the FastAPI Event Stream, each component is installed into its own
**Namespace**, which the component's playbook creates.

Although the "Operators" create their own **Namespaces**, and objects, some
also install material in the Data Manager **Namespace**. The `viz-operator`
and `jupyter-operator` do. These DM objects, typically **ConfigFiles**,
are installed separately from the operator using a `site_dm.yaml` file in the
operator's Ansible repository.

## Updating an installation
The playbooks are (or should be) idempotent. Once Squonk is installed,
running any playbook again should not damage the installation (assuming
no-one has edited any of the objects).

If you need to install a new AS version you just need to run the
corresponding `site.yaml` file again. Here we install a new version of the AS: -

```bash
ansible-playbook site.yaml -e as_installation_name=scw-scaleway -e as_image_tag=4.7.2
```

> The above assumes we've already set `ANSIBLE_VAULT_PASSWORD_FILE` and `KUBECONFIG`
