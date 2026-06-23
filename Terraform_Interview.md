# Terraform Interview Narrative — How to Explain It Out Loud

***

## "How Do You Run Terraform on AWS?"

To run Terraform on AWS, you start by installing the AWS CLI and running `aws configure` — which stores your access key, secret key, and default region in `~/.aws/credentials`. Terraform's AWS provider picks those credentials up automatically, so your provider block needs nothing more than a region. From there, you write your resource blocks — VPCs, EC2 instances, security groups — and run `terraform init` to download the `hashicorp/aws` provider plugin. `terraform plan` shows you exactly what will be created, changed, or destroyed, and `terraform apply` executes it by calling AWS APIs on your behalf. The **one thing unique to AWS** compared to the other clouds is its state locking model: you store the state file in an S3 bucket, but you also need a separate DynamoDB table with a `LockID` partition key to prevent two engineers from running `apply` simultaneously and corrupting state — that two-piece backend setup (S3 + DynamoDB) is something AWS-specific that you must provision before running `terraform init` with a remote backend. In production, teams replace static access keys with IAM role assumption, either via an EC2 instance profile or OIDC federation from a CI/CD system, so no long-lived credentials ever touch the pipeline.

***

## "How Do You Run Terraform on Azure?"

To run Terraform on Azure, you install the Azure CLI and run `az login`, which opens a browser-based authentication flow and stores a token that Terraform's `azurerm` provider reads automatically as Application Default Credentials. You then write a provider block — the **`features {}`** block is mandatory even when empty, which catches many first-timers off guard. From there you define resources like `azurerm_resource_group`, `azurerm_virtual_network`, and `azurerm_linux_virtual_machine`, and run the standard `terraform init`, `plan`, `apply` flow. The **one thing unique to Azure** is that the `azurerm` provider always requires that `features {}` block, and in CI/CD environments you authenticate via a Service Principal — a dedicated app identity in Microsoft Entra ID — whose credentials (`ARM_CLIENT_ID`, `ARM_CLIENT_SECRET`, `ARM_TENANT_ID`, `ARM_SUBSCRIPTION_ID`) are set as environment variables. State is stored in Azure Blob Storage, which uses Azure's native blob lease mechanism for locking — there is no separate lock table to provision, unlike AWS. You create a Storage Account and a container, reference them in the `backend "azurerm" {}` block, and Azure handles concurrency control automatically.

***

## "How Do You Run Terraform on GCP?"

To run Terraform on GCP, you install the Google Cloud CLI and run `gcloud auth application-default login`, which writes a credentials file to `~/.config/gcloud/application_default_credentials.json` that Terraform's `google` provider reads automatically. Your provider block specifies the GCP project ID and region, and from there you define resources like `google_compute_network`, `google_compute_subnetwork`, and `google_compute_instance`. The **one thing unique to GCP** compared to AWS and Azure is that GCP APIs must be explicitly enabled on the project before Terraform can create those resources — for example, you must run `gcloud services enable compute.googleapis.com` before any Compute Engine resource will succeed. Forgetting this step produces a confusing API-not-enabled error that has nothing to do with your Terraform code. State is stored in a GCS bucket, referenced in the `backend "gcs" {}` block with a bucket name and prefix — GCS uses object versioning and conditional writes for locking, with no separate lock table required. In CI/CD, authentication uses a Service Account key JSON file pointed to by the `GOOGLE_APPLICATION_CREDENTIALS` environment variable, or Workload Identity Federation for keyless OIDC-based auth.

***

## Terraform Similarities Across All Three Clouds

These eight things are **identical regardless of which cloud you target** — confidently state these to show you understand what is universal versus what changes per provider:

- **CLI workflow is always the same** — `terraform init` → `terraform validate` → `terraform fmt` → `terraform plan` → `terraform apply` → `terraform destroy` on every cloud without exception.
- **HCL syntax and language features are provider-agnostic** — variables, locals, outputs, expressions, `count`, `for_each`, `depends_on`, `lifecycle`, and data sources work identically across AWS, Azure, and GCP.
- **State file concept is universal** — every Terraform run produces or updates a `terraform.tfstate` JSON file that tracks the real-world identity of every managed resource, regardless of cloud.
- **Remote backend pattern is the same** — all three clouds offer a blob/object storage backend for state with some form of locking; the `backend {}` block syntax and workflow (`terraform init -reconfigure`) is identical.
- **Modules work the same way** — `module {}` blocks, input variables, and outputs behave identically; a module written for AWS can be refactored for Azure or GCP without changing module mechanics.
- **Workspaces are provider-agnostic** — `terraform workspace new staging` creates an isolated state namespace on any backend; the concept, commands, and `${terraform.workspace}` interpolation work the same everywhere.
- **Secret handling best practices are identical** — mark sensitive variables with `sensitive = true`, never commit `.tfvars` files containing secrets, and use environment variables or external secret managers regardless of cloud.
- **Provider version pinning and `required_providers` block** — the structure of `terraform { required_providers { } }` is the same for all providers; only the `source` string and version constraint value differ.

***

## 6 Multi-Cloud Terraform Interview Q&A

**Q:** How do you manage multi-cloud infrastructure in a single Terraform project?

**A:** The cleanest pattern is to use separate provider blocks — one for each cloud — within a single root module, pinning each to a specific version in `required_providers`. Resources from different providers can reference each other directly; for example, an AWS EKS cluster output can feed into a Kubernetes provider configuration in the same root module. For large-scale multi-cloud estates, the better pattern is separate Terraform configurations per cloud, with cross-cloud dependencies passed via remote state data sources using `data "terraform_remote_state"`. This keeps blast radius small, state files independent, and permissions scoped per cloud. Avoid cramming everything into one giant state file — it makes plans slow, locks painful, and rollback risky.

***

**Q:** What is a Terraform workspace and when would you use it?

**A:** A workspace is an isolated state namespace within a single Terraform configuration. Running `terraform workspace new staging` creates a separate state file so that `plan` and `apply` on the `staging` workspace never touch the `production` workspace state. The current workspace name is available as `${terraform.workspace}` for conditional logic. Workspaces are useful for lightweight environment separation in small teams where the infrastructure shape is nearly identical across environments. However, for large-scale or strongly isolated environments — especially multi-account AWS or multi-subscription Azure — the better practice is separate directories or separate repositories per environment, because workspaces still share the same code and provider configuration, which can allow a misconfiguration in one workspace to affect another.

***

**Q:** How do you handle secrets in Terraform across different clouds?

**A:** The rule is: never hardcode secrets in HCL files, never commit `.tfvars` files containing passwords, and never store secrets in outputs without marking them `sensitive = true`. In CI/CD, inject cloud credentials as environment variables scoped to the pipeline run. For application secrets that Terraform needs to create or reference — database passwords, API keys — use each cloud's native secret manager as a data source: `data "aws_secretsmanager_secret_value"`, `data "azurerm_key_vault_secret"`, or `data "google_secret_manager_secret_version"`. The Terraform state file stores all resource attributes including sensitive values, so the state backend must have encryption at rest enabled — S3 server-side encryption, Azure Blob encryption, or GCS CMEK — and access must be tightly restricted.

***

**Q:** What happens if two engineers run `terraform apply` at the same time?

**A:** Without locking, both engineers read the same state file, compute their own plans, and then both write back — the second write overwrites the first and the state no longer reflects reality. Resources may be duplicated, drift silently, or be destroyed unexpectedly. This is why state locking exists: AWS uses a DynamoDB item as a mutex; Azure Blob Storage uses a blob lease; GCS uses conditional writes. When a lock is held, the second `apply` blocks and displays a lock message with the lock holder's ID and time. If a process dies mid-apply and leaves a stale lock, you release it with `terraform force-unlock <lock-id>` — but only after confirming no apply is actually in progress.

***

**Q:** How do you refactor Terraform resources without destroying and recreating them?

**A:** There are three techniques. First, `terraform state mv <old_address> <new_address>` renames a resource in the state file without touching real infrastructure — use this when renaming resources or reorganizing module structure. Second, the `moved {}` block in HCL (Terraform 1.1+) declares a rename in code so that other users get the state migration automatically on their next `plan` without running CLI commands. Third, `terraform import <address> <real-resource-id>` brings an existing cloud resource under Terraform management by writing its attributes into state — after importing, you still need to write the matching HCL resource block. Always run `terraform plan` after any state manipulation to confirm the diff is empty before applying.

***

**Q:** What is the difference between `terraform plan -out` and running `terraform apply` directly, and why does it matter in CI/CD?

**A:** `terraform plan -out=tfplan.bin` saves the exact proposed change set to a binary file. `terraform apply tfplan.bin` executes that exact saved plan with no recalculation and no confirmation prompt. In CI/CD, this is critical for safety: the `plan` step runs in a PR or pre-merge stage for human review, and the saved plan file is passed to `apply` in the post-merge stage. This guarantees that what was reviewed is exactly what gets applied — no drift can occur between the plan review and the apply if infrastructure changed in the interval. Running `terraform apply` without a saved plan file recalculates the plan at apply time, which can produce surprises if someone else changed resources between your `plan` and your `apply`.
