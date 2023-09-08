# Docker-Build-Push-Deploy

This folder contains GitHub workflows that automate the process of Docker build, pushing to ECR registry and deployment to development and production environments in AWS.

These workflows provide structure, automation, consistency, and control to the software development and deployment process. They enhance collaboration, reduce human errors, and help ensure the reliability and stability of the applications in both development and production environments.

## Build-push-dev.yaml

This workflow builds a Docker image and publishes it to an AWS ECR repository. It also includes steps to query the AWS account ID, run a vulnerability scanner, and deploys to a dev account using Terraform. The workflow is triggered by pushing a tag to GitHub. It is triggered by a workflow call event and requires specific input parameters. The workflow runs on a self-hosted runner.

The caller workflow trigger is : on pushing a tag that has a rc  suffix.

Why create a workflow that automates docker build, push and deploy to dev process?

Automation: By creating a workflow that automate the process of building and publishing Docker images to AWS ECR it eliminates the need for manual intervention and reduces the chances of errors.
Consistency: With this workflow, you can define a standardized build process, ensuring that every Docker image follows the same steps and configurations. This helps maintain consistency across your application deployments.
Versioning: these workflows allow you to track and manage different versions of your Docker images, making it easier to roll back to a previous version if needed.
Scalability: As the project grows, having an automated workflow for building and publishing Docker images simplifies the process and allows you to scale your infrastructure without manual bottlenecks.

Inputs:
oidc_role (required): The OIDC role to assume for configuring AWS credentials.
region (required): The AWS region where the ECR repository is located.
terraform_dir (required): The directory path of the Terraform files.
ecr_repo_name (required): The name of the ECR repository.

Make sure that these inputs are provided in the caller workflow.



Workflow steps:

Checkout repo code: Checks out the repository code using the actions/checkout action. The ref parameter specifies the branch to check out.

Query account ID: Queries the AWS account ID using the AWS CLI and stores it in the environment variable $GITHUB_OUTPUT to be used at a later step.

Docker build: Builds the Docker image and tags it with the AWS ECR repository details and the tag that triggered the workflow. It logs in to the ECR repository using the AWS CLI, builds the image, and stores the image reference in the $GITHUB_OUTPUT variable.

Run Trivy vulnerability scanner: Uses the aquasecurity/trivy-action to scan the Docker image for vulnerabilities. It specifies parameters such as image reference, output format, severity level, etc.

Docker push: Pushes the Docker image to the ECR repository using the docker push command.

Configure AWS credentials for dev ENV: Configures AWS credentials for the development environment using the aws-actions/configure-aws-credentials action. It assumes a specific role (oidc_role) in the development account and sets the AWS region.


Setup Terraform: Sets up Terraform using the hashicorp/setup-terraform action. It specifies the Terraform version.

Terraform apply: Applies the Terraform configuration, using the terraform apply command. It sets the image release value based on the tag in dev.tfvars file and applies the changes.

Push changes automatically: Automatically commits and pushes changes to the repository. It uses the EndBug/add-and-commit action, specifying the commit message, author details, and email.

### Sample caller workflow:
```
name: build-push-dev

on:
  push:
    tags:
      - '[0-9].[0-9].[0-9]rc*'
      - '[0-9].[0-9].[0-9][0-9]rc*'
      - '[0-9].[0-9][0-9].[0-9]rc*'
      - '[0-9][0-9].[0-9].[0-9]rc*'
      - '[0-9].[0-9][0-9].[0-9][0-9]rc*'
      - '[0-9][0-9].[0-9].[0-9][0-9]rc*'
      - '[0-9][0-9].[0-9][0-9].[0-9]rc*'
      - '[0-9][0-9].[0-9][0-9].[0-9][0-9]rc*'

permissions: read-all

jobs:
  build_push-dev:
    permissions: 
      id-token: write
      contents: write
    uses: boldlink/workflows-and-pipelines/.github/workflows/gh001-build-push-dev.yaml@main
    with:
      oidc_role: arn:aws:iam::<insert_dev_account_number>:role/<insert-oidc_role_name>
      region:<insert_aws_region>
      terraform_dir: "${GITHUB_WORKSPACE}/<insert_terraform_directory_path>"
      ecr_repo_name:<insert_ecr_repo_name>
    secrets: inherit
```

## release.yaml

This workflow automates the release process by creating a github release, building and pushing a Docker image to ecr, performing vulnerability scanning, and deploying to a dev account using Terraform. It is triggered by a workflow call event and requires specific input parameters. The workflow runs on a self-hosted runner.

In the caller workflow, release.yaml is only called when code is pushed to the default branch. This is because we do not want to create software releases if the code is not well tested or is broken.

Why create a workflow that automates version release?
Release Management: By creating a workflow specifically for creating releases, we establish a systematic approach to managing and tracking different versions of your software. This helps in maintaining an organized and structured development process.
Testing and Validation: The workflow includes steps to deploy the release to a development account in AWS, allowing you to thoroughly test and validate the application in an environment similar to the production setup.
Isolated Environment: Deploying to a development account ensures that the release is isolated from the production environment, reducing the risk of impacting the live system with untested changes.


Inputs:
Config-name (required): The name of the configuration file for release-drafter/release-drafter@v5 action.
Oidc_role (required): The  OIDC role in dev account to assume for configuring AWS credentials.
Region (required): The AWS region where the ECR repository is located.
Terraform_dir (required): The directory path of the Terraform files.
Ecr_repo_name (required): The name of the ECR repository.

### Workflow Steps:

Get Merged Pull Request: Uses the actions-ecosystem/action-get-merged-pull-request action to get information about the merged pull request. It requires the GITHUB_TOKEN secret.

Release Drafter: Uses the release-drafter/release-drafter action to create a release. It creates a release only if the merged pull request does not have the label 'no-release'. It specifies parameters such as publishing the release, setting it as a pre-release or not, and the configuration name.

Checkout Repo Code: Checks out the repository code using the actions/checkout action. The ref parameter specifies the branch to check out.

Query Account ID: Queries the AWS account ID using the AWS CLI and stores it in the $GITHUB_OUTPUT environment variable. 

Docker Build: Builds the Docker image and tags it with the AWS ECR repository details and the release tag name. It logs in to the ECR repository using the AWS CLI, builds the image, and stores the image reference in the $GITHUB_OUTPUT variable. 

Run Trivy Vulnerability Scanner: Uses the aquasecurity/trivy-action to scan the Docker image for vulnerabilities. It specifies parameters such as the image reference, output format, severity level, etc. This step is skipped if the merged pull request has the label 'no-release'.

Docker Push: Pushes the Docker image to the ECR repository using the docker push command. 

Configure AWS Credentials for Dev ENV: Configures AWS credentials for the development environment using the aws-actions/configure-aws-credentials action. It assumes the specified oidc_role and sets the AWS region. 

Confirm Logged-in User: Uses the AWS CLI to confirm the logged-in user's identity. 

Setup Terraform: Sets up Terraform using the hashicorp/setup-terraform action. 

Terraform Apply: Applies the Terraform configuration. It changes the image_release value in the env/${ENV}.tfvars file based on the release tag name, initializes Terraform, applies the changes to the infrastructure, and formats the Terraform files. 

Create Pull Request: Creates a pull request automatically to update the image release in the dev.tfvars file. It uses the peter-evans/create-pull-request action, specifying parameters such as the token, author details, commit message, branch name, base branch, and PR title and body. 

NOTE: The steps above are skipped if the pull request contains the label ‘no-release’

### Sample caller Workflow:
```
name: Release

on:
  push:
    branches:
      - main
      

permissions: read-all

jobs:
  release:
    permissions:
      contents: write
      pull-requests: write
      id-token: write
    uses: boldlink/workflows-and-pipelines/.github/workflows/gh001-release.yaml@main
    with:
      config-name:  workflows/config/rd-config.yaml
      oidc_role: arn:aws:iam::<insert_dev_account_number>:role/<insert-oidc_role_name>
      region:<insert_aws_region>
      terraform_dir: "${GITHUB_WORKSPACE}/<insert_terraform_directory_path>"
      ecr_repo_name:<insert_ecr_repo_name>
    secrets: inherit
```


## deploy-prod.yaml

This workflow automates the deployment of the application to the production environment using Terraform. It is triggered by a workflow call event and requires specific input parameters. The workflow runs on a self-hosted runner

It’s caller workflow is manually triggered using the "workflow_dispatch" event.


Why create a workflow that deploys to production?
Controlled Deployment: By creating a separate workflow for production deployments, you can enforce stricter controls and approvals to ensure that only verified and validated releases are deployed to the production environment.


Inputs:
oidc_role (required): The OIDC role to assume for configuring AWS credentials in production account.
region (required): The AWS region where the Terraform resources are deployed.
terraform_dir (required): The directory path of the Terraform files.


Steps:
Install jq: Installs the jq utility using the sudo yum install command to process JSON data in step 2.

Get Latest Release: Retrieves the latest release tag name from the GitHub repository using the GitHub API. It stores the tag name in the $GITHUB_OUTPUT environment variable.
Display Latest Tag: Displays the latest release tag name in the workflow log.

Checkout Repo Code: Checks out the repository code using the actions/checkout action. The ref parameter specifies the branch to checkout.

Configure AWS Credentials for Prod ENV: Configures AWS credentials for the production environment using the aws-actions/configure-aws-credentials action. It assumes the specified oidc_role and sets the AWS region.

Setup Terraform: Sets up Terraform using the hashicorp/setup-terraform action. It specifies the Terraform version as ${TF_VER}.

Terraform Apply: Applies the Terraform configuration. It changes the image_release value in the prod.tfvars file based on the latest release tag name, initializes Terraform, applies the changes to the infrastructure, and formats the Terraform files.

Create Pull Request: Creates a pull request automatically to update the image release in the prod.tfvars file. It uses the peter-evans/create-pull-request action, specifying parameters such as the token, author details, commit message, branch name, base branch, and PR title and body.


### Sample caller workflow:
```
name: deploy-prod

on:
  workflow_dispatch:

permissions: read-all

jobs:
  deploy-prod:
    permissions: 
      id-token: write
      contents: write
    uses: boldlink/workflows-and-pipelines/.github/workflows/gh001-deploy-prod.yaml@main
    with:
      oidc_role: arn:aws:iam::<insert_dev_account_number>:role/<insert-oidc_role_name>
      region:<insert_aws_region>
      terraform_dir: "${GITHUB_WORKSPACE}/<insert_terraform_directory_path>"
      ecr_repo_name:<insert_ecr_repo_name>
    secrets: inherit
```

## Github Environments

We use Environments to describe a general deployment target like production and development. When a GitHub Actions workflow deploys to an environment, the environment is displayed on the main page of the repository.

You can configure environments with protection rules and secrets. When a workflow job references an environment, the job won't start until all the environment's protection rules pass. A job also cannot access secrets that are defined in an environment until all the environment protection rules pass. We set protection rules for prod environment to ensure that the workflow is reviewed and approved first before being deployed.

For more information on how to create and delete a GitHub environment, see here: https://docs.github.com/en/actions/deployment/targeting-different-environments/using-environments-for-deployment

