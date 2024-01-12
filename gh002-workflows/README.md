# Workflows to Build Docker Images, Push to AWS ECR and Deploy to both Development and Production Environments
## Overview
This directory contains GitHub Actions workflows for automating Docker build and push to AWS ECR, as well as deployment to development and production environments using Terraform.

- Docker build

- Docker Push to ECR
This step grabs the latest tag from the development branch that trigger the workflow and use the tag for pushing the image to AWS ECR

- Deploy to Development Environment
This step deploys the pushed image to development environment for testing purposes. This is to ensure that the image is not deployed to production environment incase there is any issue

- Deploy to Production
After successful test in the development environment, the image is now deployed to the production Environment

**NOTE:** These are not reusable workflows

## Docker Build and Push Workflow
### Workflow Description
The `docker-build-and-push` workflow is triggered on push events with tags matching the pattern `[0-99]+.[0-99]+.[0-99]+rc[0-9]+`. It builds a Docker image, pushes it to AWS ECR, runs vulnerability scanning, image linting, and deploys the application to an AWS ECS environment using Terraform.

## Workflow Steps
1. **Checkout Code**: Fetches the repository code.
2. **Configure AWS Credentials**: Configures AWS credentials using the OIDC role specified.
3. **Login to ECR**: Authenticates with the AWS ECR registry.
4. **Get Latest Tag Pushed**: Retrieves the latest tag pushed to the repository.
5. **Tag Image for ECR**: Tags the Docker image for Amazon ECR.
6. **Set up Docker Buildx**: Sets up Docker Buildx for multi-platform builds.
7. **Build Docker Image**: Builds the Docker image using the Dockerfile in the specified source directory.
8. **Run Trivy Vulnerability Scanner**: Scans the Docker image for vulnerabilities using Trivy.
9. **Run Dockle Image Linter**: Performs linting on the Docker image using Dockle.
10. **Push Docker Image to ECR**: Pushes the Docker image to the specified AWS ECR repository.
11. **Setup Terraform**: Sets up Terraform for the deployment.
12. **Deploy Image to AWS ECS using Terraform**: Deploys the Docker image to AWS ECS using Terraform.
13. **Persist Image Value Changes in Variables.tf**: Commits changes to the Terraform variables file.

## Release and Deploy to Development Workflow
### Workflow Description
The `Release and Deploy to Dev` workflow is triggered upon the completion of the Release workflow. It builds and pushes the Docker image, deploys the application to the development environment using Terraform, and creates a pull request to persist the release tag in the image.

### Workflow Steps
1. **Checkout Code**: Fetches the repository code.
2. **Configure AWS Credentials**: Configures AWS credentials using the OIDC role specified.
3. **Login to ECR**: Authenticates with the AWS ECR registry.
4. **Get Latest Release**: Retrieves the latest release tag.
5. **Tag Image for ECR**: Tags the Docker image for Amazon ECR using the release tag.
6. **Set up Docker Buildx**: Sets up Docker Buildx for multi-platform builds.
7. **Build Docker Image**: Builds the Docker image using the Dockerfile in the specified source directory.
8. **Push Docker Image to ECR**: Pushes the Docker image to the specified AWS ECR repository.
9. **Setup Terraform**: Sets up Terraform for the deployment.
10. **Deploy Image to AWS ECS using Terraform**: Deploys the Docker image to AWS ECS using Terraform.
11. **Create PR to Persist Image**: Creates a pull request to persist the release tag in the Docker image.

## Deploy to Production Workflow
### Workflow Description
The `Deploy to Prod` workflow is triggered manually using the GitHub Actions interface. It deploys the application to the production environment using Terraform.

### Workflow Steps
1. **Checkout Code**: Fetches the repository code.
2. **Configure AWS Credentials**: Configures AWS credentials using the OIDC role specified.
3. **Setup Terraform**: Sets up Terraform for the deployment.
4. **Deploy Image to AWS ECS using Terraform**: Deploys the Docker image to AWS ECS in the production environment.

## Environment Variables
The following are the variables used

- **ECR_REPOSITORY**: Name of the AWS ECR repository.
- **TF_VERSION**: Terraform version.
- **AWS_REGION**: AWS region.
- **OIDC_ROLE**: ARN of the IAM role for OIDC.
- **TF_DIR**: Path to the directory containing Terraform files.
- **SRC_DIR**: Path to the directory containing the Dockerfile.
- **ENV**: Environment to deploy the application.
- **DEPLOY_ENV**: Environment to persist the image (used in the dev workflow).

Provide these values in the following `ENV` section of the workflows

```yaml
name: docker-build-and-push

on:
  push:
    tags:
      - '[0-99]+.[0-99]+.[0-99]+rc[0-9]+'

permissions:
  id-token: write # This permission is required to authorize the request for the GitHub OIDC token used by the configure-aws-credentials action
  contents: write  # This is required for actions/checkout & committing changes back to the PR

env:
  ECR_REPOSITORY: <name_of_ecr_repository>
  TF_VERSION: 0.14.11
  AWS_REGION: <aws_region_here>
  OIDC_ROLE: arn:aws:iam::<provide_account_ID_here>:role/<name_of_the_iam_role>
  TF_DIR: ./terraform #path to directory containing terraform files
  SRC_DIR: ./src #path to directory containing Dockerfile
  ENV: dev #environment to deploy the application to
```
## Permissions
The workflows require write permissions for id-token and contents, and specific permissions for the OIDC token in the `docker-build-and-push` workflow.

## BOLDLink-SIG 2024
