# Changelog
All notable changes to this project will be documented in this file.

## [ unreleased ]
- fix: regular expression used as a trigger on push tags that have a `rc*` suffix
- Workflows don't start in self-hosted runner for manually triggered workflows that have an approval step. The self-hosted runner is [terraform-aws-github-runner](https://github.com/philips-labs/terraform-aws-github-runner). The affected workflow is `deploy.yaml`

## [ released ]
- feat: Github workflow that deploys to AWS dev environment automatically when specific tags are pushed to github
- feat: Github workflow that deploys to AWS dev environment automatically when code is merged to main(default branch)
- feat: Github workflow that deploys to AWS prod environment when it is manually triggered.