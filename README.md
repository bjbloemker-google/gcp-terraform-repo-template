# GCP Terraform Project Template

Welcome! This repository serves as a comprehensive template to kickstart your Google Cloud Platform (GCP) infrastructure projects using Terraform. It includes pre-configured GitHub Actions workflows for automated deployment leveraging Workload Identity Federation for secure, keyless authentication.

## Quickstart

Get started in just a few steps:

1.  **Use this template:**
    *   Click the "Use this template" button on the GitHub repository page.
    *   Select "Create a new repository."
    *   Choose an owner and repository name for your new project.
    *   Click "Create repository from template."
2.  **Clone your new repository:**
    ```bash
    git clone https://github.com/YOUR_USERNAME/YOUR_NEW_REPO_NAME.git
    cd YOUR_NEW_REPO_NAME
    ```
3.  **Configure Prerequisites:** Follow the steps in the "Prerequisites" section below.
4.  **Customize:** Adapt the Terraform files to define your desired infrastructure as outlined in "Customizing the Template."
5.  **Deploy:** Push your changes to `main` to trigger the `plan` workflow, and then push a tag (e.g., `v1.0.0`) to trigger the `apply` workflow.

## Prerequisites

Before deploying, ensure the following are set up in your GCP project and GitHub repository:

1.  **GCS Bucket for Terraform State:**
    *   Create a Google Cloud Storage (GCS) bucket to store the Terraform state file (`.tfstate`). This is crucial for managing your infrastructure's state, especially in a team environment.
    *   Enable versioning on this bucket to keep a history of your state files, allowing for rollbacks if needed.

2.  **Workload Identity Federation (WIF):**
    *   **Workload Identity Pool and Provider:** Create a Workload Identity Pool and a Provider within that pool. The provider must be configured to trust GitHub Actions. The "issuer URI" for GitHub is `https://token.actions.githubusercontent.com`.
    *   **Service Account (SA):** Create a new Google Cloud Service Account or choose an existing one. This SA will be what Terraform uses to make changes to your GCP resources. Grant it the necessary IAM roles (e.g., `roles/editor` for broad permissions, or more granular roles like `roles/compute.admin`, `roles/storage.admin` for the state bucket, etc.) based on the resources you intend to manage.
    *   **Grant SA Impersonation Rights:** Allow your GitHub Actions workflow to impersonate the Service Account. Grant the `roles/iam.workloadIdentityUser` role on the Service Account to the principal representing your GitHub repository. This principal typically looks like: `principalSet://iam.googleapis.com/projects/<YOUR_PROJECT_NUMBER>/locations/global/workloadIdentityPools/<YOUR_POOL_ID>/attribute.repository/<YOUR_GITHUB_ORG_OR_USER>/<YOUR_GITHUB_REPO>`.

3.  **GitHub Repository Secrets:**
    *   The GitHub Actions workflows require specific secrets to be configured in your repository settings (Settings > Secrets and variables > Actions > New repository secret). These secrets are essential for authentication and configuration:
        *   `TERRAFORM_STATE_FILE_BUCKET`: The name of the GCS bucket created for Terraform state.
        *   `WIF_PROVIDER`: The full identifier of your Workload Identity Provider.
        *   `WIF_SERVICE_ACCOUNT`: The email address of the Service Account that Terraform will impersonate.
    *   Detailed explanations of these secrets are also available in the "GitHub Actions Workflows" section.

4.  **`gcloud` CLI (Optional but Recommended):**
    *   Install and configure the Google Cloud SDK (`gcloud` CLI) on your local machine. This is useful for manually setting up the above resources or interacting with your GCP project. Installation instructions can be found [here](https://cloud.google.com/sdk/docs/install).

Once these prerequisites are in place, you can customize the template and deploy your infrastructure.

## Repo Structure

This template is organized to promote clarity and ease of management:

*   `.github/workflows/`: Contains the GitHub Actions CI/CD workflow definitions (`plan.yaml` and `apply.yaml`).
*   `backend.tf`: Configures Terraform's remote state backend (GCS bucket). **Note:** This file is commented out by default. You must uncomment it and update the bucket name.
*   `main.tf`: The primary file where you will define your GCP resources using Terraform.
*   `providers.tf`: Declares necessary Terraform providers, primarily the Google Cloud provider. You can specify provider versions here.
*   `terraform.auto.tfvars`: Used to automatically load variable values for your Terraform configuration (e.g., `project_id`, `region`).
*   `variables.tf`: Declares input variables for your Terraform configuration, including types, descriptions, and default values.

## GitHub Actions Workflows

The CI/CD pipelines in `.github/workflows/` automate your Terraform deployments using Workload Identity Federation.

### `plan.yaml` Workflow

*   **Trigger:** Automatically runs on pushes to the `main` branch.
*   **Purpose:** Generates a Terraform execution plan to preview infrastructure changes. **This workflow does not alter your GCP infrastructure.**
*   **Key Steps:**
    1.  Checks out code.
    2.  Authenticates to GCP using WIF (via `WIF_PROVIDER` and `WIF_SERVICE_ACCOUNT` secrets).
    3.  Sets up Terraform (version specified by `TERRAFORM_VERSION` in the workflow, e.g., `1.8.0`).
    4.  Runs `terraform init` (configures backend with `TERRAFORM_STATE_FILE_BUCKET` secret).
    5.  Runs `terraform plan` (uses variables from `TERRAFORM_VAR_FILE`, e.g., `terraform.auto.tfvars`) and outputs a plan artifact.

### `apply.yaml` Workflow

*   **Trigger:** Runs when a new tag (e.g., `v1.0.0`, `release-v2.1`) is pushed.
*   **Purpose:** Applies the Terraform configuration to your GCP environment. **This workflow actively modifies your GCP infrastructure.**
*   **Key Steps:**
    1.  Checks out code corresponding to the tag.
    2.  Authenticates to GCP using WIF.
    3.  Sets up Terraform.
    4.  Runs `terraform init`.
    5.  Runs `terraform apply` (using variables from `TERRAFORM_VAR_FILE`, with `-auto-approve`).

### Required GitHub Secrets (Recap)

Ensure these are configured in your repository settings for the workflows:

*   `TERRAFORM_STATE_FILE_BUCKET`: Name of your GCS bucket for Terraform state.
*   `WIF_PROVIDER`: Full identifier of your Workload Identity Provider (e.g., `projects/<PROJECT_NUMBER>/locations/global/workloadIdentityPools/<POOL_ID>/providers/<PROVIDER_ID>`).
*   `WIF_SERVICE_ACCOUNT`: Email of the Service Account for Terraform to impersonate (e.g., `terraform-sa@<PROJECT_ID>.iam.gserviceaccount.com`).

The workflows also use environment variables like `TERRAFORM_VERSION` (which you can update in the YAML files) and `TERRAFORM_VAR_FILE`.

## GCP Authentication: Workload Identity Federation

This template uses Google Cloud's Workload Identity Federation (WIF) for secure, passwordless authentication from GitHub Actions to GCP.

### Overview of WIF

WIF allows external workloads (like GitHub Actions) to impersonate a GCP Service Account without using long-lived service account keys. It uses short-lived OIDC tokens exchanged for Google access tokens.

*   **Benefits:** Enhanced security (no static keys) and simplified management.

### WIF in this Template

The `google-github-actions/auth` action in the workflows handles WIF authentication using the `WIF_PROVIDER` and `WIF_SERVICE_ACCOUNT` secrets.

### Setup Guidance (High-Level)

1.  Create a Workload Identity Pool in GCP.
2.  Create a Provider within the pool, configured to trust GitHub Actions (issuer: `https://token.actions.githubusercontent.com`).
3.  Ensure your Service Account has the necessary IAM roles.
4.  Grant the GitHub repository (`principalSet://...`) the `roles/iam.workloadIdentityUser` role on the Service Account.

**Refer to the official Google Cloud documentation for detailed WIF setup instructions:**
*   [Configuring Workload Identity Federation](https://cloud.google.com/iam/docs/workload-identity-federation)
*   [Setting up WIF for GitHub Actions](https://cloud.google.com/iam/docs/configuring-workload-identity-federation#github)

## Customizing the Template

Adapt this template for your specific infrastructure needs:

### 1. Defining Resources (`main.tf`)

*   **Action:** Populate `main.tf` with Terraform HCL code describing your desired GCP infrastructure (VMs, GCS buckets, VPCs, etc.).
*   **Example:**
    ```terraform
    # Example: Define a GCS bucket in main.tf
    resource "google_storage_bucket" "my_bucket" {
      name     = "my-unique-app-bucket-${var.project_id}"
      location = var.region
      uniform_bucket_level_access = true
    }
    ```

### 2. Managing Variables (`variables.tf` & `terraform.auto.tfvars`)

*   **`variables.tf`:** Declare input variables (e.g., `project_id`, `region`) with types, descriptions, and optional defaults.
    ```terraform
    variable "project_id" {
      description = "The GCP project ID."
      type        = string
    }
    variable "region" {
      description = "The GCP region for resources."
      type        = string
      default     = "us-central1"
    }
    ```
*   **`terraform.auto.tfvars`:** Provide values for variables declared in `variables.tf`.
    ```tfvars
    project_id = "your-gcp-project-id"
    region     = "us-east1"
    ```
*   **Sensitive Data:** For sensitive values, do not commit them to `terraform.auto.tfvars`. Instead, use GitHub Secrets and pass them as environment variables (e.g., `TF_VAR_my_secret_var`) in your workflow files.

### 3. Configuring State Backend (`backend.tf`)

*   **Action:**
    1.  Open `backend.tf`.
    2.  Uncomment the `terraform { backend "gcs" { ... } }` block.
    3.  Update `bucket` to your GCS bucket name (must match `TERRAFORM_STATE_FILE_BUCKET` secret).
    ```terraform
    terraform {
      backend "gcs" {
        bucket  = "your-terraform-state-bucket-name" # MUST match TERRAFORM_STATE_FILE_BUCKET secret
        prefix  = "terraform/state"
      }
    }
    ```

### 4. Provider Configuration (`providers.tf`)

*   **Action:** Review `providers.tf`. The Google provider uses variables like `var.project_id` and `var.region`. You can pin the provider version for stability.
    ```terraform
    provider "google" {
      project = var.project_id
      region  = var.region
      # version = "~> 4.80.0" # Example: Pin to a specific version range
    }
    ```

### 5. General Advice

*   **Review All Files:** Familiarize yourself with the template's structure.
*   **Incremental Changes:** Start small, test with `plan` and `apply` workflows, then gradually expand.
*   **Documentation:** Consult official [Terraform](https://www.terraform.io/docs) and [Google Provider](https://registry.terraform.io/providers/hashicorp/google/latest/docs) documentation.
