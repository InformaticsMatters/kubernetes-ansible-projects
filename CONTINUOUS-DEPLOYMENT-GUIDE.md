# Continuous Deployment (via AWX)
We can use [AWX] as a playbook server to run the Squonk component installation
playbooks.

>   Although the AWX project is no longer bring developed we still rely on it.
    Now that we have worked hard to ensure that the playbooks can be executed from
    the command-line we will have to assess our needs for the replacement
    when (if) it becomes available.

To do utilise AWX we create **Projects** for each Ansible repo,
and a **Run Template** for each installation that uses the corresponding **Project**
and its `site.yaml` **Playbook**.

In the **Run Template** `Variables` section we set just two variables: an image tag
(i.e. `as_image_tag`), and the installation name (i.e. `as_installation_name`).

> By setting the **Prompt on launch** checkbox for the `Variables` in the
  **Run Template** we ensure that we can inject new image tag values from the
  CI process that triggers execution of the **Run Template**.

Finally, in order to run the playbook against the cluster we provide
two credentials in the **Run Template**: a **Kubernetes Bearer Token** to access the
cluster, and a **Vault** to store the vault key for the installation.

With this done we can then use our **awx-trigger** logic from within a GitHub **Action**
or the **GitLab** CI to run the chosen **Run Templates** when a new component image
is available.

We tend to configure CD for the _main_ components (AS, DM and, UI) but often
run the operator playbooks outside of AWX because the operators rarely change.
We can put operator playbooks (**Projects**) and configure **Run Templates**
if we wish, we just don't.

## Example GitHub Action
We have a simple _trigger_ action that can be used from within a GitHub Action Job.
You provide the **Run Template** you want to execute, the URL of the AWX server,
a valid username and password (for an account on the AWX server that can execute the
**Run Template**) and the variable name and value the playbook uses to set the
tag of the component image.

```yaml
trigger-awx-test:
  runs-on: ubuntu-latest
  environment: awx/dls-test
  steps:
  - name: Trigger AWX test
    uses: informaticsmatters/trigger-awx-action@v3
    with:
      template: Squonk/2 Data Manager UI -test-
      template-host: https://awx.example.com
      template-user: ${{ secrets.AWX_USER }}
      template-user-password: ${{ secrets.AWX_USER_PASSWORD }}
      template-var: ui_image_tag
      template-var-value: ${{ needs.release.outputs.new_release_version }}
```

## Example GitLab CI
The following snippet illustrates the use of triggering an AWX **Job Template** from
within GitLab CI. It looks complicated but what you see here are excerpts from two
files: the base logic (`.ax-trigger`) and the use of it.

```yaml
variables:
  TRIGGER_AWX_ORIGIN: https://raw.githubusercontent.com/informaticsmatters/trigger-awx
  TRIGGER_AWX_VERSION: 2.2.0

.awx-trigger:
  image: python:3.10.19
  before_script:
  - python --version
  - pip install --upgrade pip
  - echo "TRIGGER_AWX_ORIGIN=${TRIGGER_AWX_ORIGIN}"
  - echo "TRIGGER_AWX_VERSION=${TRIGGER_AWX_VERSION}"
  - >
    curl --location --retry 3
    ${TRIGGER_AWX_ORIGIN}/${TRIGGER_AWX_VERSION}/requirements.txt
    --output trigger-awx-requirements.txt
  - pip install -r trigger-awx-requirements.txt
  - >
    curl --location --retry 3
    ${TRIGGER_AWX_ORIGIN}/${TRIGGER_AWX_VERSION}/trigger-awx-tag.sh
    --output trigger-awx-tag.sh
  - chmod +x trigger-awx-tag.sh

staging deploy:
  stage: deploy
  extends: .awx-trigger
  variables:
    AWX_HOST: https://awx.example.com
    AWX_USER: gitlab
    AWX_USER_PASSWORD: ${AWX_USER_PASSWORD_STAGING}
    AWX_TEMPLATE: Squonk/2 Data Manager API -test-
    AWX_TRIGGER_VARIABLE: dt_image_tag
  script:
  - ./trigger-awx-tag.sh "$CI_COMMIT_TAG" $AWX_TRIGGER_VARIABLE "$AWX_TEMPLATE"
  environment:
    name: staging
    deployment_tier: staging
    url: "$DEPLOYMENT_URL"
```

---

[awx]: https://github.com/ansible/awx
