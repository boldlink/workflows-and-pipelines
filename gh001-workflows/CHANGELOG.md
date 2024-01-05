# Changelog
All notable changes to this project will be documented in this file.

## [ unreleased ]
- Workflows don't start in self-hosted runner for manually triggered workflows that have an approval step. The self-hosted runner is [terraform-aws-github-runner](https://github.com/philips-labs/terraform-aws-github-runner). The affected workflow is `deploy.yaml`
- feat: workflow steps that push images to other container registries; docker hub,gcr etc.

## [ released ]
- feat: Github workflow that builds and pushed docker images to an ecr registry
- feat: a Github workflow that creates a repository release and attaches a docker image as a release asset
- feat: a Github workflow that added a label to pull requests based on the PR branch prefix
- feat: Github workflow that deploys to an AWS account automatically when a tag that conforms to the following regex format `[0-9]+.[0-9]+.[0-9]+rc[0-9]+` is pushed to Github
