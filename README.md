# Azure Pipeline templates

This is a collection of various templates used in build, test and deploy of various code repositories. Depending on how code should be built or deployed different stages, jobs or steps can be included from project.

## Setup

To use the templates a service connection towards Github has to be created. In `azure-pipelines.yaml` add a reference to this repository.

```yaml
resources:
  repositories:
  - repository: templates
    type: github
    name: avinor/azure-pipelines-templates
    endpoint: endpoint-name
```

To target a specific version of the pipelines add for instance `ref: refs/tags/v1.0`.

## Templates

In most cases only the stages templates have to be included from `azure-pipelines.yaml` file. It is also possible to use the jobs and steps directly, but stages will define most common scenarios.

### terraform.yaml

`stages/terraform.yaml` defines a template to plan and apply terraform templates using terragrunt. It takes a list of paths that are configured as jobs in Azure Pipeline. In each path it will first run `terragrunt plan-all` and then `terragrunt apply-all`.

```yaml
resources:
  repositories:
  - repository: templates
    type: github
    name: avinor/azure-pipelines-templates
    endpoint: endpoint-name

stages:
- template: stages/terraform.yaml@templates
  parameters:
    azureSubscription: 'subscription-name'
    environment: env-name
    sasToken: sas-token
    paths:
    - name: WestEurope
      displayName: West Europe
      path: path-for-resources/westeurope
```

Stage template takes 4 input parameters:

| Parameter | Description
|-----------|------------
| `azureSubscription` | Name of the service connection to use for the AzureCLI task. Need to have permissions to do the deployment
| `environment` | Name of environment to deploy to. Test, stage, prod etc
| `sasToken` | SAS Token to access storage account for shared state. This is a secret
| `paths` | A list of jobs to add. Each taking `name`, `displayName` and `path`. First 2 defining job properties and last the path in repository where resources to deploy exist.

### terraform-modules.yaml

Terraform modules runs some simple validations on a terraform module. It will look for all examples folders in the repository and run validation on examples:

```bash
terraform init -backend=false examples/...
terraform validate examples/...
```

It will always set backend to false to make sure that modules that define terraform configuration does not fail.

```yaml
resources:
  repositories:
  - repository: templates
    type: github
    name: avinor/azure-pipelines-templates
    endpoint: endpoint-name

stages:
- template: stages/terraform-modules.yaml@templates
```