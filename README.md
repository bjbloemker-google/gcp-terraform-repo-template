# gcp-terraform-template

This repo exist as a template to serve as the base of your application. It contains the required files for a GCP terraform project and associated Github actions for deployment using Workload Identity Federation.

## Repo Structure

**Deployment files:**

- `.github/workflows`: Base Github Actions workflow for GCP deployment

**Terraform files:**

- `backend.tf`: defines state file backend
- `providers.tf`: declare terraform providers that will be used
- `terraform.auto.tfvars`: input values for terraform code
- `variables.tf`: terraform variable declaration

