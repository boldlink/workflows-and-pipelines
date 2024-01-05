# Workflows to Build Docker Images, Push to AWS ECR and Deploy to both Development and Production Environments
This directory contains the github Workflows that automate the following processes;
- Docker build

- Docker Push to ECR
This step grabs the latest tag from the development branch that trigger the workflow and use the tag for pushing the image to AWS ECR

- Deploy to Development Environment
This step deploys the pushed image to development environment for testing purposes. This is to ensure that the image is not deployed to production environment incase there is any issue

- Deploy to Production
After successful test in the development environment, the image is now deployed to the production Environment

**NOTE:** These are not reusable workflows


## BOLDLink-SIG 2024
