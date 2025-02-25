# Terraform Orb [![CircleCI Orb Version](https://img.shields.io/badge/endpoint.svg?url=https://badges.circleci.io/orb/ovotech/terraform)](https://circleci.com/orbs/registry/orb/ovotech/terraform)

This orb can be used to plan and apply terraform modules.
It is published as `ovotech/terraform@1.6`

**Note: this orb only supports terraform `<=0.14`.**

**Version 2 of this orb is now available for terraform `>=0.14`, published as a
separate orb, `terraform-v2`. Head over [here](../terraform-v2/v1-to-v2.md)
for help upgrading.**

## Executors

The orb provides executors for running terraform commands.
An executor defines the docker image to use for jobs.

### default

The executor named `default` is the same as `terraform-0_11`

### terraform-0_11

[Dockerfile](executor/Dockerfile-0.11)

This executor uses terraform 0.11.14 by default. However `tfswitch` is
installed to allow you to adjust at runtime.
See [Specifying a terraform version](https://github.com/ovotech/circleci-orbs/tree/master/terraform#specifying-a-terraform-version)
to change this.

It also contains:
- helm + terraform-provider-helm
- terraform-provider-acme
- google-cloud-sdk
- aws-cli
- ovo's terraform-provider-aiven as version 0.0.1
- ovo's kafka user provider

If the AIVEN_PROVIDER environment variable is set, also has:
- Aiven's terraform-provider-aiven from versions 1.0.0+

### terraform-0_12

[Dockerfile](executor/Dockerfile-0.12)

This executor uses terraform 0.12.29 by default. However `tfswitch` is
installed to allow you to adjust at runtime.
See [Specifying a terraform version](https://github.com/ovotech/circleci-orbs/tree/master/terraform#specifying-a-terraform-version)
to change this.

It also contains:
- Aiven's terraform-provider-aiven
- google-cloud-sdk
- Helm 2 available as `helm2` and `helm`
- Helm 3+ available as `helm3` See [Using Helm 3](https://github.com/ovotech/circleci-orbs/tree/master/terraform#Using-Helm-3)
- aws-cli
- ovo's kafka user provider

### terraform-0_12-slim

[Dockerfile](executor/Dockerfile-0.12-slim)

A much much slimmer image. Carries one version of each provider/tool.
This executor uses terraform 0.12.29 by default. However `tfswitch` is
installed to allow you to adjust at runtime.
See [Specifying a terraform version](https://github.com/ovotech/circleci-orbs/tree/master/terraform#specifying-a-terraform-version)
to change this.

It also contains:
- Aiven's terraform-provider-aiven
- google-cloud-sdk
- Helm 2 available as `helm2` and `helm`
- Helm 3+ available as `helm3` See [Using Helm 3](https://github.com/ovotech/circleci-orbs/tree/master/terraform#Using-Helm-3)
- aws-cli
- ovo's kafka user provider

### terraform-0_13

[Dockerfile](executor/Dockerfile-0.13)

This executor uses terraform 0.13.0 by default. However `tfswitch` is
installed to allow you to adjust at runtime.
See [Specifying a terraform version](https://github.com/ovotech/circleci-orbs/tree/master/terraform#specifying-a-terraform-version)
to change this.

It also contains:
- Aiven's terraform-provider-aiven
- google-cloud-sdk
- Helm 2 available as `helm2` and `helm`
- Helm 3+ available as `helm3` See [Using Helm 3](https://github.com/ovotech/circleci-orbs/tree/master/terraform#Using-Helm-3)
- aws-cli
- stable "ovo" provider with ovo_kafka_user resource (`source = "terraform.ovotech.org.uk/pe/ovo"`)
- beta "aiven-kafka-user" provider with aiven-kafka-users_user resource that enables auto-rotation of credentials  (`source = "terraform.ovotech.org.uk/pe/aiven-kafka-users"`)


## Commands

Terraform can obtain credentials from environment variables set in
circleci.

For the AWS provider, set the AWS_ACCESS_KEY and AWS_SECRET_ACCESS_KEY
environment variables.

For the gcloud provider, set GCLOUD_SERVICE_KEY to be a GCP service
account key as a base64 encoded or plain text string. You can also
optionally set GOOGLE_PROJECT_ID and GOOGLE_COMPUTE_ZONE.

If GITHUB_USERNAME and GITHUB_TOKEN environment variables are set,
the `plan` command will add a comment on an open PR with the plan.

By default, when using the `apply` command the plan must have been approved
by being merged from a PR that has had a comment added by a previous `plan` command.
If the plan is not found or has drifted, then the `apply` command will fail.

This is to ensure that the orb only applies changes that have been reviewed by a human.

You can disable this behaviour by setting `auto_approve: true` in the `apply` step,
which will always apply any terraform changes.

Available commands:
- plan
- apply
- output
- check
- destroy
- new-workspace
- destroy-workspace
- fmt-check
- validate
- version
- in-workspace
- publish-module

### plan

This command runs the terraform plan command.

Parameters:

- path (string): Path the the terraform module to run the plan in
- workspace (string): Terraform workspace to run the command in (default: 'default')
- label (string): An optional friendly name for the environment this plan is for. This must be set if there are multiple plans in a job with the same path and workspace.
- backend_config_file (string): Comma separated list of terraform backend config files. File location is relative to either checkout dir or directory specified in `path`.
- backend_config (string): Comma separated list of backend configs, e.g. foo=bar
- var_file (string): Comma separater list of terraform var files. File location is relative to either checkout dir or directory specified in `path`.
- var (string): Comma separated list of vars to set, e.g. foo=bar
- parallelism (int): Limit the number of concurrent operations
- add_github_comment (bool): 'true' to comment on an open PR with the plan. Default: true
- use_chdir (bool): 'true' to use the -chdir option in terraform. This is only relevant to terraform 0.14 and onward and is for compatibility with provider lockfiles. Default: false

### apply

This command runs the terraform apply command.

Parameters:

- path: Path the the terraform module to run the plan in
- workspace: Terraform workspace to run the command in (default: 'default')
- label: An optional friendly name for the environment this apply is for. This must be the same as the label of the corresponding plan command.
- backend_config_file: Comma separated list of terraform backend config files
- backend_config: Comma separated list of backend configs, e.g. foo=bar
- target: Comma separated list of targets to apply against, e.g. kubernetes_secret.tls_cert_public,kubernetes_secret.tls_cert_private NOTE: this argument only takes effect if auto_approve is also set.
- var_file: Comma separater list of terraform var files
- var: Comma separated list of vars to set, e.g. foo=bar
- auto_approve: true, to apply the plan, even if it has not been approved through a PR.
- parallelism: Limit the number of concurrent operations
- output_path (string): An optional path to write a json file containing the output variables.
- use_chdir (bool): 'true' to use the -chdir option in terraform. This is only relevant to terraform 0.14 and onward and is for compatibility with provider lockfiles. Default: false

### output

This command saves the output variables from a terraform state into a json file.

Parameters:

- path (string): Path the the terraform module to get the outputs for
- workspace (string): Terraform workspace to run the command in (default: 'default')
- backend_config_file (string): Comma separated list of terraform backend config files
- backend_config (string): Comma separated list of backend configs, e.g. foo=bar
- output_path (string): The path to write the json file containing the output variables.
- use_chdir (bool): 'true' to use the -chdir option in terraform. This is only relevant to terraform 0.14 and onward and is for compatibility with provider lockfiles. Default: false

### check

This command runs the terraform plan command, and fails the build if any
changes are required. This is intended to run on a schedule to notify if
manual changes to your infrastructure have been made.

Parameters:

- path: Path the the terraform module to run the plan in
- workspace: Terraform workspace to run the command in (default: 'default')
- backend_config_file: Comma separated list of terraform backend config files
- backend_config: Comma separated list of backend configs, e.g. foo=bar
- var_file: Comma separater list of terraform var files
- var: Comma separated list of vars to set, e.g. foo=bar
- parallelism: Limit the number of concurrent operations
- use_chdir (bool): 'true' to use the -chdir option in terraform. This is only relevant to terraform 0.14 and onward and is for compatibility with provider lockfiles. Default: false

### destroy

This runs the terraform destroy command, destroying all resources.

Parameters:

- path: Path the the terraform module to destroy the resource in
- workspace: Terraform workspace to run the command in (default: 'default')
- backend_config_file: Comma separated list of terraform backend config files
- backend_config: Comma separated list of backend configs, e.g. foo=bar
- var_file: Comma separater list of terraform var files
- var: Comma separated list of vars to set, e.g. foo=bar
- parallelism: Limit the number of concurrent operations
- use_chdir (bool): 'true' to use the -chdir option in terraform. This is only relevant to terraform 0.14 and onward and is for compatibility with provider lockfiles. Default: false

### new-workspace

This creates a new terraform workspace

Parameters:

- path: Path to the terraform module to create a workspace in
- workspace: Terraform workspace to create
- backend_config_file: Comma separated list of terraform backend config files
- backend_config: Comma separated list of backend configs, e.g. foo=bar
- use_chdir (bool): 'true' to use the -chdir option in terraform. This is only relevant to terraform 0.14 and onward and is for compatibility with provider lockfiles. Default: false

### destroy-workspace

This destroys all resource in a workspace and deletes the workspace

Parameters:

- path: Path to the terraform module to destroy a workspace in
- workspace: Terraform workspace to destroy
- backend_config_file: Comma separated list of terraform backend config files
- backend_config: Comma separated list of backend configs, e.g. foo=bar
- var_file: Comma separater list of terraform var files
- var: Comma separated list of vars to set, e.g. foo=bar
- parallelism: Limit the number of concurrent operations
- use_chdir (bool): 'true' to use the -chdir option in terraform. This is only relevant to terraform 0.14 and onward and is for compatibility with provider lockfiles. Default: false

### fmt-check

Check that the terraform files in a directory are in canonical format,
as output by `terraform fmt`. This command will fail if any file is
in non-canonical format.

Parameters:

- path: Path to the directory to check

### validate

Statically validates the terraform configuration in a directory.

Parameters:

- path: Path to the terraform configuration to validate
- use_chdir (bool): 'true' to use the -chdir option in terraform. This is only relevant to terraform 0.14 and onward and is for compatibility with provider lockfiles. Default: false

### in-workspace

Initialize a terraform working directory and execute steps in it.
The steps parameter is a nested list of steps to execute.

Parameters:

- path: Path the the terraform module to create a workspace in
- workspace: Terraform workspace to destroy
- backend_config_file: Comma separated list of terraform backend config files
- backend_config: Comma separated list of backend configs, e.g. foo=bar
- steps: The steps to execute in the initialized working directory
- use_chdir (bool): 'true' to use the -chdir option in terraform. This is only relevant to terraform 0.14 and onward and is for compatibility with provider lockfiles. Default: false

### version

Prints terraform and provider versions.

Parameters:

- path: Path to the terraform configuration to print versions for

### publish-module

This publishes a terraform module to a terraform module registry.

These environment variables should be set:
- TF_REGISTRY_HOST: The hostname of the registry to publish to.
- TF_REGISTRY_TOKEN: The registry api token to use.

Parameters:

- path: Path to the terraform module to publish
- module_name: The full module name, of the form "$NAMESPACE/$NAME/$PROVIDER"
- version_file_path: Path to a file containing the semantic version to publish.

## Jobs

This orb contains the jobs:
- plan
- apply
- check
- destroy
- new-workspace
- destroy-workspace

These jobs run their respective command in the default executor
(which uses terraform 0.11). The jobs have the same parameters as the commands.

## Examples

### A simple example

In this example a plan for the module in tf/ is generated and attached
to the open PR. If that PR is then merged, the plan is applied.

```yaml

version: 2.1

orbs:
  terraform: ovotech/terraform@1.6

workflows:
  test:
    jobs:
      - terraform/plan:
          path: tf
          filters:
            branches:
              ignore: master

      - terraform/apply:
          path: tf
          filters:
            branches:
              only: master
```

### A real-world example

This configuration defines it's own plan and apply jobs which use the
orb's plan and apply commands on multiple terraform modules. It uses
terraform 0.12 via `terraform-0_12` executor.
It also configures a helm repo within the the container for use with the
terraform helm provider.

```yaml

version: 2.1

orbs:
  terraform: ovotech/terraform@1.6

jobs:
  terraform_plan:
    executor: terraform/terraform-0_12
    steps:
    - checkout
    - run:
        name: Add helm repo
        command: |
          echo $GOOGLE_SERVICE_ACCOUNT | base64 -d > /tmp/google_creds
          export GOOGLE_APPLICATION_CREDENTIALS=/tmp/google_creds
          gcloud auth activate-service-account --key-file=/tmp/google_creds
          helm plugin install https://github.com/nouney/helm-gcs --version 0.1.4
          helm repo add gauges gs://gauges-helm-repo/

    - terraform/plan:
        path: terraform/deployments/cluster
        workspace: gauges-uat
    - terraform/plan:
        path: terraform/deployments/cluster-init
        workspace: gauges-uat
    - terraform/plan:
        path: terraform/deployments/gauges-uat

    - terraform/plan:
        path: terraform/deployments/cluster
        workspace: gauges-prd
    - terraform/plan:
        path: terraform/deployments/cluster-init
        workspace: gauges-prd
    - terraform/plan:
        path: terraform/deployments/gauges-prd

    - terraform/plan:
        path: terraform/deployments/sqs-test

  terraform_apply:
    executor: terraform/terraform-0_12
    steps:
    - checkout
    - run:
        name: Add helm repo
        command: |
          echo $GOOGLE_SERVICE_ACCOUNT | base64 -d > /tmp/google_creds
          export GOOGLE_APPLICATION_CREDENTIALS=/tmp/google_creds
          gcloud auth activate-service-account --key-file=/tmp/google_creds
          helm plugin install https://github.com/nouney/helm-gcs --version 0.1.4
          helm repo add gauges gs://gauges-helm-repo/

    - terraform/apply:
        path: terraform/deployments/cluster
        workspace: gauges-uat
    - terraform/apply:
        path: terraform/deployments/cluster-init
        workspace: gauges-uat
    - terraform/apply:
        path: terraform/deployments/gauges-uat

    - terraform/apply:
        path: terraform/deployments/cluster
        workspace: gauges-prd
    - terraform/apply:
        path: terraform/deployments/cluster-init
        workspace: gauges-prd
    - terraform/apply:
        path: terraform/deployments/gauges-prd

    - terraform/apply:
        path: terraform/deployments/sqs-test

workflows:
  commit:
    jobs:
      - terraform_plan:
          filters:
            branches:
              ignore: master
      - terraform_apply:
          filters:
            branches:
              only: master

```

### Using Helm 3
#### Repositories
If you are using Helm 3 (terraform provider > v1) you must explicitly add
repositories, this includes charts in `stable` which is no longer included 
in Helm by default. 

[Official Documentation](https://helm.sh/docs/intro/quickstart/#initialize-a-helm-chart-repository)

This can be done using the `helm3` command prior to any apply jobs.

```yaml
  terraform_apply:
    executor: terraform/terraform-0_12
    steps:
    - checkout
    - run:
        name: Add helm repo
        command: |
            helm3 repo add stable https://kubernetes-charts.storage.googleapis.com
            helm3 repo update

    - terraform/apply:
        path: terraform/deployments/cluster
        workspace: gauges-uat
```

#### Default version
The `helm` command will use `helm2` by default, however if the `HELM` 
environment variable is set to `helm3`, the `helm` command invokes Helm 3 instead.

### Using the aiven provider

When using the default or terraform-0_11 executors, OVO's
`terraform-provider-aiven` is available at version `0.0.1`.
To use Aiven's `terraform-provider-aiven` set the AIVEN_PROVIDER
environment variable and set a version equal or greater than `1.0.0` in
the provider configuration.

When using the terraform-0_12 executor Aiven's `terraform-provider-aiven` is
always available. (And OVO's is not).

```yaml

version: 2.1

orbs:
  terraform: ovotech/terraform@1.6

jobs:
  terraform_plan:
    executor: terraform/default
    environment:
      AIVEN_PROVIDER: true
    steps:
      - checkout
      - terraform/plan:
          path: tf

workflows:
  test:
    jobs:
      - terraform_plan:
          filters:
            branches:
              ignore: master
```

### Checking for changes

This examples checks the infrastructure every morning. If changes are
detected to any of the terraform resources the build is failed.

```yaml

version: 2.1

orbs:
  terraform: ovotech/terraform@1.6

workflows:
  nightly:
    triggers:
      - schedule:
          cron: "0 8 * * *"
          filters:
            branches:
              only:
                - master
    jobs:
      - terraform/check:
          path: prod

```

## GitHub

To attach the plan to a PR, create the GITHUB_USERNAME and GITHUB_TOKEN
environment variables in the CircleCI project. This should be the username
(not email address) and a Personal Access Token of a github user that has access to
the repo. The token requires the `repo, write:discussion` scopes.

It's recommended to enable **"Only build pull requests"**  in your CircleCI 
config when using this setting. If not enabled this could lead to a creation
of a PR after the CircleCI job has run, which means the Plan comment cannot be
added. Settings can be found under "Advanced Settings" e.g.
https://circleci.com/gh/ovotech/$YOUR_REPO/edit#advanced-settings

To make best use of this orb, require that the plan is always reviewed
before merging the PR to approve. You can enforce this in github by
going to the branch settings for the repo and enable protection for
the master branch:

1. Enable 'Require pull request reviews before merging'
1. Check 'Dismiss stale pull request approvals when new commits are pushed'
1. Enable 'Require status checks to pass before merging'
1. Select the 'ci/circleci: terraform_plan' check.
1. Enable 'Require branches to be up to date before merging'
1. In the CircleCI project advanced settings, enable 'Only build pull requests'.

## Private Terraform Module Registries

You can use this orb with private Terraform Module registries.

To specify the registry api token, set TF_REGISTRY_HOST and
TF_REGISTRY_TOKEN environment variables in the CircleCI settings.

## Masking of sensitive values

The terraform commands use [tfmask](https://github.com/cloudposse/tfmask) to mask sensitive output for these resources:

- random_id
- kubernetes_secret
- acme_certificate

You can customise this list using the TFMASK_RESOURCES_REGEX environment variable. See the tfmask docs for details.
Please create a github issue to suggest additional resources that need masking.

## Specifying a terraform version

The version of terraform to use is discovered from the first of:
1. A [`required_version`](https://www.terraform.io/docs/configuration/terraform.html#specifying-a-required-terraform-version)
   constraint in the terraform configuration.
2. A [tfswitch](https://warrensbox.github.io/terraform-switcher/) `.tfswitchrc` file
3. A [tfenv](https://github.com/tfutils/tfenv) `.terraform-version` file in path of the terraform
   configuration.
4. Terraform v0.12.5 for the 0.12 executor, or 0.11.14 for the 0.11 executor.

The `required_version` constraint goes somewhere in your terraform configuration:
```hcl
terraform {
  required_version = "0.12.28"
}
```

tfswitch and tfenv make it easy to install the correct version locally.  
Their config files contain a terraform version number to use:
```
0.12.18
```
