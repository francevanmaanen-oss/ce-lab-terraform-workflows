# Lab M4.10 - Terraform Git Workflows

## Overview
This lab implements a professional Terraform CI/CD workflow using GitHub Actions.
Feature branches, automated validation, plan output on PRs, and apply on merge.

## Workflow
1. Create feature branch
2. Make infrastructure changes
3. Push and open PR
4. GitHub Actions runs format, validate, plan
5. Review plan output in PR comment
6. Merge to main

## CI/CD Pipeline
- **Format check** — ensures consistent code style
- **Validate** — catches syntax errors
- **Plan** — shows proposed changes on PRs

## Repository Structure
├── .github/
│   ├── workflows/terraform.yml
│   └── pull_request_template.md
├── .gitignore
├── main.tf
├── variables.tf
└── outputs.tf

## Infrastructure
This lab provisions the following AWS resources:
- **S3 bucket** — with versioning enabled
- **S3 bucket encryption** — AES256 server-side encryption
- **S3 public access block** — all public access blocked

## Issues Encountered & Fixes

### 1. Missing `variables.tf`
The initial project structure was missing `variables.tf`, causing VS Code's
Terraform extension to flag unresolved variable references (`var.aws_region`,
`var.project_name`, `var.environment`) in `main.tf`. Fixed by creating the file
with the required variable declarations.

### 2. Terraform version — expired GPG key
The lab specified Terraform `1.6.0` in the GitHub Actions workflow. This version
fails to install the AWS provider because HashiCorp's GPG signing key used in
that release has since expired: Error while installing hashicorp/aws v5.100.0: error checking signature: openpgp: key expired

Fixed by bumping the Terraform version to `1.12.0` in `.github/workflows/terraform.yml`.

### 3. Plan output capture
The original workflow relied on `steps.plan.outputs.stdout` to capture plan output
for the PR comment, which is unreliable with newer versions of the
`hashicorp/setup-terraform` action. Fixed by explicitly piping output using `tee`
and the `$GITHUB_OUTPUT` file mechanism.

### 4. GitHub CLI required but not documented
The lab uses `gh repo create` and `gh pr create` without mentioning that the
GitHub CLI must be installed and authenticated first. Run `gh --version` to check,
install from https://cli.github.com if needed, and authenticate with `gh auth login`.

### 5. AWS credentials required
The `terraform plan` step requires AWS credentials. Without adding
`AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` as repository secrets, the plan
step fails silently and the PR comment will be empty. Add these under
**Settings → Secrets and variables → Actions** in your repository before running
the workflow.