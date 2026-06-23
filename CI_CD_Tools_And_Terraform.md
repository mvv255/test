# Senior Multi-Cloud DevOps/SRE Interview Prep — Sections 5 & 6

***

## SECTION 5 — CI/CD Tools Comparison Table

| **Area** | **Jenkins** | **GitHub Actions** | **GitLab CI/CD** | **Azure DevOps Pipelines** | **CircleCI** | **ArgoCD** |
|---|---|---|---|---|---|---|
| **Definition & Core Philosophy** | Open-source automation server; extensible plugin model; runs anything anywhere | Event-driven workflows tightly integrated with GitHub repositories | CI/CD built directly into GitLab platform; DevOps from source to deploy | Microsoft-hosted DevOps pipeline for code, builds, tests, and deployments | Developer-first cloud CI with fast Docker-based pipelines | GitOps CD tool for Kubernetes; declarative desired-state sync from Git |
| **Configuration File Name & Format** | `Jenkinsfile` — Groovy DSL (declarative or scripted) | `.github/workflows/*.yml` — YAML | `.gitlab-ci.yml` — YAML | `azure-pipelines.yml` — YAML | `.circleci/config.yml` — YAML | No pipeline file; app is described in Kubernetes manifests + `Application` CRD in Git |
| **Pipeline Trigger Mechanism** | Webhooks, SCM polling, cron schedules, manual triggers, upstream job chains | GitHub events: push, pull_request, schedule, workflow_dispatch, release | Git push, merge request, schedule, API, pipeline triggers | Push, PR, schedule, manual, Azure Repos or GitHub events | Push, PR, schedule, API, manual approval | Continuous Git sync; reconciles cluster state to Git on every commit; manual or auto sync |
| **Build Agent / Runner Type** | Self-hosted agents (master + worker nodes); any OS/environment | GitHub-hosted runners (Ubuntu, Windows, macOS) or self-hosted | GitLab shared runners (Docker/Kubernetes executors) or self-managed | Microsoft-hosted or self-hosted agents | Cloud-hosted Docker containers or self-hosted | No build agent — ArgoCD runs inside the Kubernetes cluster and uses kubectl/Helm/Kustomize |
| **Parallelism Support** | Parallel stages in declarative pipeline; matrix build via plugins | `matrix` strategy and `jobs` run in parallel natively | `parallel:` keyword; DAG with `needs:`; split testing | `parallel` jobs; matrix strategy; stage parallelism | `parallelism` key per job; fan-in/fan-out workflows | N/A — sync operations are declarative; progressive delivery via Argo Rollouts |
| **Secret Management approach** | Jenkins Credentials Store; `withCredentials()` block; integrates with HashiCorp Vault | GitHub Encrypted Secrets (Actions Secrets); environment secrets; OIDC for cloud auth | CI/CD Variables: masked/protected; Group/Project level; integrates with Vault | Azure Key Vault linked variable groups; pipeline secrets; OIDC federation | Environment variables; Contexts (org/project level); OIDC support | Kubernetes Secrets, Sealed Secrets, External Secrets Operator, or Vault Agent |
| **Docker / Kubernetes Integration** | Docker plugin for building images; `kubectl` CLI in pipeline; requires manual setup | Native docker/login/build/push actions; OIDC to ECR/GCR/ACR; kubectl action available | Built-in Docker-in-Docker support; integrated Kubernetes deploy steps | Docker build/push tasks; Azure Container Registry tasks; AKS deploy tasks | First-class Docker orbs; Kubernetes deploy orbs; native DinD | Kubernetes-native; syncs Helm/Kustomize/raw manifests directly to cluster |
| **Artifact Storage** | Built-in artifact archive; integrates with Nexus, Artifactory, S3 | GitHub Actions cache and artifacts (uploaded with upload-artifact action) | GitLab Package Registry, Container Registry, Generic Packages | Azure Artifacts (NuGet, npm, Maven, Python, Universal packages) | Workspaces; integrates with S3, registries, Artifactory | Git repository is the artifact; Helm chart versions and image tags in manifests |
| **Pricing Model** | Open-source; free to self-host; compute cost is your own infrastructure | Free tier: 2000 min/month; paid per minute over limit; free for public repos | Free tier on GitLab.com; SaaS tiers; self-managed free CE or paid EE | Free tier: 1800 min/month; paid beyond; free for public projects | Free dev plan; usage-based credit pricing; paid performance plans | Open-source; free to self-install on Kubernetes; Argo CD enterprise via Akuity |
| **Setup Complexity (1–5)** | **5/5** — most complex; requires server setup, plugin management, and maintenance | **1/5** — no server; create a workflow YAML, commit, done | **2/5** — built into GitLab; minimal config for hosted; slightly more for self-managed | **2/5** — wizard-driven; tight Azure ecosystem integration; easy onboarding | **2/5** — SaaS-first; fast setup; self-hosted adds complexity | **3/5** — requires Kubernetes cluster to install; ArgoCD CLI + web UI; concept overhead |
| **Best For (use case / team size)** | Large enterprises with custom legacy build chains, mixed technology, existing investment; any size | Small to large teams using GitHub; modern cloud-native apps; open-source projects | Teams using GitLab as their full DevOps platform from code to deploy; mid to large | Microsoft-centric organizations; .NET, Azure, Teams integrations; enterprise compliance | Startups to mid-size; speed-focused teams; microservices; Docker-native workflows | Kubernetes-first teams doing GitOps; platform teams managing multiple clusters |
| **Key Concept to explain in interview** | Shared Library — reusable Groovy code in a separate repo referenced across all Jenkinsfiles | OIDC federation — how GitHub Actions can authenticate to AWS/Azure/GCP without storing static secrets | Pipeline-as-code with DAG — how `needs:` creates a dependency graph instead of sequential stages | Azure Pipelines YAML templates — reusable stages/jobs defined in shared template files | Orbs — reusable shareable workflow packages (CircleCI's equivalent of GitHub Actions' reusable workflows) | App of Apps pattern — one parent ArgoCD Application that manages many child Applications across namespaces or clusters |

***

### Jenkins — Working Pipeline Example

```groovy
// Jenkinsfile — Declarative Pipeline
// Builds Docker image, pushes to ECR, deploys to EKS

pipeline {
    agent any

    environment {
        AWS_REGION       = 'us-east-1'
        ECR_REGISTRY     = '123456789012.dkr.ecr.us-east-1.amazonaws.com'
        IMAGE_NAME       = 'myapp'
        IMAGE_TAG        = "${env.BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/myorg/myapp.git'
            }
        }

        stage('Unit Tests') {
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }

        stage('Docker Build') {
            steps {
                script {
                    dockerImage = docker.build("${ECR_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}")
                }
            }
        }

        stage('Push to ECR') {
            steps {
                withCredentials([
                    [$class: 'AmazonWebServicesCredentialsBinding',
                     credentialsId: 'aws-ecr-credentials',
                     accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                     secretKeyVariable: 'AWS_SECRET_ACCESS_KEY']
                ]) {
                    sh """
                        aws ecr get-login-password --region ${AWS_REGION} | \
                        docker login --username AWS --password-stdin ${ECR_REGISTRY}
                        docker push ${ECR_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}
                        docker tag ${ECR_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG} \
                                   ${ECR_REGISTRY}/${IMAGE_NAME}:latest
                        docker push ${ECR_REGISTRY}/${IMAGE_NAME}:latest
                    """
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                withKubeConfig([credentialsId: 'eks-kubeconfig']) {
                    sh """
                        kubectl set image deployment/myapp \
                          myapp=${ECR_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG} \
                          -n production
                        kubectl rollout status deployment/myapp -n production
                    """
                }
            }
        }
    }

    post {
        success {
            slackSend channel: '#deployments', message: "Deploy SUCCESS: ${IMAGE_NAME}:${IMAGE_TAG}"
        }
        failure {
            slackSend channel: '#deployments', message: "Deploy FAILED: ${IMAGE_NAME}:${IMAGE_TAG}"
        }
    }
}
```

#### How Jenkins Pipeline Concepts Work

**Stages** are logical groupings of steps inside a `pipeline {}` block. Each `stage` appears as a column in the Jenkins Blue Ocean UI. Stages can be parallelized using the `parallel {}` block.

**Environment variables** are declared in the `environment {}` block at the pipeline or stage level. They are exposed as shell environment variables inside `sh` steps. Dynamic values like `env.BUILD_NUMBER` and `env.GIT_COMMIT` are injected automatically.

**Secrets** are stored in the Jenkins Credentials Store and accessed via `withCredentials()`. The credentials are injected into environment variables scoped only within the `withCredentials` block — they are masked in logs. For cloud credentials, use cloud provider plugins (AWS Credentials Plugin) rather than plain text bindings.

***

### GitHub Actions — Working Pipeline Example

```yaml
# .github/workflows/build-push-deploy.yml
name: Build, Push, and Deploy

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

env:
  AWS_REGION: us-east-1
  ECR_REGISTRY: 123456789012.dkr.ecr.us-east-1.amazonaws.com
  IMAGE_NAME: myapp

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up JDK 21
        uses: actions/setup-java@v4
        with:
          java-version: '21'
          distribution: 'temurin'

      - name: Run tests
        run: mvn test

  build-push:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    outputs:
      image_tag: ${{ steps.meta.outputs.tags }}

    permissions:
      id-token: write      # required for OIDC
      contents: read

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Configure AWS credentials via OIDC
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::123456789012:role/github-actions-ecr
          aws-region: ${{ env.AWS_REGION }}

      - name: Login to Amazon ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v2

      - name: Build and push Docker image
        id: meta
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: |
            ${{ env.ECR_REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }}
            ${{ env.ECR_REGISTRY }}/${{ env.IMAGE_NAME }}:latest

  deploy:
    needs: build-push
    runs-on: ubuntu-latest
    environment: production   # requires manual approval if configured

    steps:
      - name: Configure AWS credentials via OIDC
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::123456789012:role/github-actions-eks
          aws-region: ${{ env.AWS_REGION }}

      - name: Update kubeconfig for EKS
        run: aws eks update-kubeconfig --region ${{ env.AWS_REGION }} --name my-eks-cluster

      - name: Deploy to EKS
        run: |
          kubectl set image deployment/myapp \
            myapp=${{ env.ECR_REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }} \
            -n production
          kubectl rollout status deployment/myapp -n production
```

#### How GitHub Actions Pipeline Concepts Work

**Jobs** are the top-level parallelizable units in a workflow. The `needs:` keyword creates a dependency so `build-push` only starts after `test` passes. Each job runs on a fresh runner unless you explicitly cache or pass data using `upload-artifact`/`download-artifact`.

**Environment variables** are declared at workflow, job, or step level using `env:`. They can reference GitHub context values like `github.sha`, `github.ref`, and `github.actor`. Use `${{ env.VAR_NAME }}` in YAML expressions and `$VAR_NAME` inside `run:` shell steps.

**Secrets** are stored in repository or organization-level Secrets in GitHub Settings and accessed as `${{ secrets.MY_SECRET }}`. The recommended production pattern is OIDC federation: GitHub Actions requests a short-lived token from AWS/Azure/GCP using the `id-token: write` permission, eliminating static credentials entirely.

***

### 8 Senior CI/CD Interview Q&A

**Q1:** What is the difference between Continuous Integration, Continuous Delivery, and Continuous Deployment?  
**A:** CI is automated build and test on every commit. Continuous Delivery produces a deployable artifact on every green build but requires manual gate to production. Continuous Deployment goes further and automatically pushes every green build to production with no human approval step.

**Q2:** How do you prevent secrets from leaking in CI/CD pipelines?  
**A:** Never store secrets in environment files or source code. Use the platform's native secrets store. Use OIDC federation where possible to issue short-lived tokens instead of static credentials. Audit pipeline logs to ensure secrets are masked. Rotate credentials regularly and scope them to minimum necessary permissions.

**Q3:** How would you implement a multi-environment promotion pipeline?  
**A:** Use separate environment stages (dev → staging → prod) where promotion requires either automated smoke tests passing or a manual approval gate. Artifact is built once and the same image tag is promoted across environments. Environment-specific config is injected at deploy time from a secrets or config store, not baked into the image.

**Q4:** What is the difference between a Jenkins Shared Library and a GitHub Actions Reusable Workflow?  
**A:** A Jenkins Shared Library is a Groovy code library referenced across Jenkinsfiles for common logic (build, push, notify). A GitHub Actions Reusable Workflow is a YAML workflow file that other workflows can call using `uses:`. Both solve the DRY problem for pipeline code — Shared Libraries are more code-like and powerful; Reusable Workflows are simpler and YAML-native.

**Q5:** How does ArgoCD differ from a traditional push-based CD tool like Spinnaker or Jenkins deploy stages?  
**A:** ArgoCD is pull-based and GitOps-driven. The cluster continuously watches a Git repository and reconciles to match it. Traditional push-based CD tools send deployment commands from outside the cluster. The GitOps model means Git is the source of truth, rollbacks are Git reverts, and audit history is a git log.

**Q6:** How do you handle a database migration in a CI/CD pipeline safely?  
**A:** Use a migration tool like Flyway or Liquibase, run migrations as a pre-deploy job with a rollback plan for each migration, ensure migrations are backward compatible with the running version of the app during rolling deployments, and gate the rollout on migration success.

**Q7:** What strategies exist for zero-downtime deployments?  
**A:** Rolling update — replaces old pods gradually. Blue-Green — two identical environments; switch traffic all at once. Canary — route a percentage of traffic to the new version and expand based on metrics. Feature flags — deploy code with flags off, enable progressively. All require readiness probes and a load balancer aware of pod health.

**Q8:** How do you detect and prevent supply chain attacks in a CI/CD pipeline?  
**A:** Pin third-party actions, Docker base images, and dependencies to specific commit SHAs or digests rather than mutable tags. Use image vulnerability scanning (Trivy, Snyk) before pushing. Sign artifacts with Sigstore/Cosign. Enable SBOM generation. Use branch protection, require signed commits, and enforce least-privilege permissions on CI runners.

***

## SECTION 6 — Terraform Across Cloud Providers

| **Area** | **Terraform + AWS** | **Terraform + Azure** | **Terraform + GCP** |
|---|---|---|---|
| **Provider Block Syntax** | `provider "aws" { region = var.aws_region }` | `provider "azurerm" { features {} subscription_id = var.subscription_id }` | `provider "google" { project = var.project_id region = var.region }` |
| **Authentication Method** | AWS credentials via `~/.aws/credentials`, `AWS_ACCESS_KEY_ID`/`AWS_SECRET_ACCESS_KEY` env vars, EC2 instance profile, or OIDC-assumed IAM role | Service Principal with client secret or client certificate, managed identity (on Azure VMs), Azure CLI (`az login`), OIDC workload identity federation | Service Account key JSON, Application Default Credentials via `gcloud auth application-default login`, Workload Identity Federation |
| **CLI Prerequisites** | AWS CLI installed and `aws configure` run with access key, secret, and default region | Azure CLI installed and `az login` run; or service principal env vars set (`ARM_CLIENT_ID`, etc.) | Google Cloud CLI installed and `gcloud auth application-default login` run; or `GOOGLE_APPLICATION_CREDENTIALS` pointing to service account JSON |
| **Core Provider Name & Registry Source** | `hashicorp/aws` from `registry.terraform.io/hashicorp/aws` | `hashicorp/azurerm` from `registry.terraform.io/hashicorp/azurerm` | `hashicorp/google` from `registry.terraform.io/hashicorp/google` |
| **Compute Resource Example** | `resource "aws_instance" "web" { ... }` | `resource "azurerm_linux_virtual_machine" "web" { ... }` | `resource "google_compute_instance" "web" { ... }` |
| **Storage Resource Example** | `resource "aws_s3_bucket" "data" { ... }` | `resource "azurerm_storage_account" "data" { ... }` + `azurerm_storage_container` | `resource "google_storage_bucket" "data" { ... }` |
| **Networking Resource Example** | `resource "aws_vpc" "main" { ... }` + `aws_subnet`, `aws_internet_gateway` | `resource "azurerm_virtual_network" "main" { ... }` + `azurerm_subnet` | `resource "google_compute_network" "main" { ... }` + `google_compute_subnetwork` |
| **IAM Resource Example** | `resource "aws_iam_role" "app" { ... }` + `aws_iam_role_policy_attachment` | `resource "azurerm_role_assignment" "app" { ... }` + `azuread_service_principal` | `resource "google_service_account" "app" { ... }` + `google_project_iam_member` |
| **State Backend Options** | S3 bucket + DynamoDB table for locking; `backend "s3" {}` | Azure Blob Storage container; `backend "azurerm" {}` | Google Cloud Storage bucket; `backend "gcs" {}` |
| **Workspace Support** | Full workspace support; different state keys in S3 per workspace | Full workspace support; different state blobs in Azure Blob per workspace | Full workspace support; different state objects in GCS per workspace; `gcs` backend stores state as `<prefix>/<workspace>.tfstate` |
| **Key Terraform Commands** | `terraform init -backend-config="bucket=my-tf-state"` then `plan`, `apply`, `destroy`; use `-var-file=prod.tfvars` | `terraform init -reconfigure` after backend change; `ARM_*` env vars for auth; `plan`, `apply`, `destroy` | `terraform init -backend-config="bucket=my-tf-state"`; set `GOOGLE_PROJECT`; `plan`, `apply`, `destroy`; use `--impersonate-service-account` for least-privilege |
| **Common Gotchas / Errors** | Forgetting to create the DynamoDB lock table before using S3 backend; `NoCredentialsError` when env vars not set; IAM permission boundary blocking resource creation | `azurerm` provider requires `features {}` block even if empty or Terraform errors on init; subscription must be set or `az account set` run first; resource group must exist before most resources | `google_project_services` can take several minutes to activate; billing must be enabled on the project or compute resources fail; some resources need `depends_on` explicit ordering |
| **Best Use Case** | AWS-first environments, EKS clusters, multi-account organizations, Lambda infrastructure, legacy EC2 automation | Azure-first enterprises, AKS clusters, Microsoft hybrid identity environments, regulated industries | GCP-first data platform teams, GKE clusters, BigQuery + data infrastructure, organizations standardized on Google Workspace |

***

### Terraform Provider Deep-Dive: AWS

#### Full Working Minimal Example

```hcl
# aws_main.tf — Create a VPC and EC2 instance

terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }

  # Optional: Remote state in S3
  # backend "s3" {
  #   bucket         = "my-tf-state-bucket"
  #   key            = "dev/ec2/terraform.tfstate"
  #   region         = "us-east-1"
  #   dynamodb_table = "terraform-locks"
  #   encrypt        = true
  # }
}

variable "aws_region" {
  type    = string
  default = "us-east-1"
}

provider "aws" {
  region = var.aws_region
}

# Look up the latest Amazon Linux 2023 AMI
data "aws_ami" "amazon_linux" {
  most_recent = true
  owners      = ["amazon"]

  filter {
    name   = "name"
    values = ["al2023-ami-*-x86_64"]
  }
}

resource "aws_vpc" "main" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_support   = true
  enable_dns_hostnames = true

  tags = {
    Name = "dev-vpc"
  }
}

resource "aws_subnet" "public" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.1.0/24"
  availability_zone       = "${var.aws_region}a"
  map_public_ip_on_launch = true

  tags = {
    Name = "dev-public-subnet"
  }
}

resource "aws_internet_gateway" "igw" {
  vpc_id = aws_vpc.main.id
  tags   = { Name = "dev-igw" }
}

resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id
  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.igw.id
  }
}

resource "aws_route_table_association" "public" {
  subnet_id      = aws_subnet.public.id
  route_table_id = aws_route_table.public.id
}

resource "aws_security_group" "web_sg" {
  name   = "web-sg"
  vpc_id = aws_vpc.main.id

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

resource "aws_instance" "web" {
  ami                    = data.aws_ami.amazon_linux.id
  instance_type          = "t3.micro"
  subnet_id              = aws_subnet.public.id
  vpc_security_group_ids = [aws_security_group.web_sg.id]

  tags = {
    Name = "dev-web-server"
  }
}

output "instance_public_ip" {
  value = aws_instance.web.public_ip
}

output "vpc_id" {
  value = aws_vpc.main.id
}
```

#### Step-by-Step: Running Terraform on AWS

1. **Install Terraform CLI**
   ```bash
   # Linux (Rocky/RHEL)
   sudo dnf install -y dnf-plugins-core
   sudo dnf config-manager --add-repo https://rpm.releases.hashicorp.com/RHEL/hashicorp.repo
   sudo dnf install terraform -y
   terraform version
   ```

2. **Install and configure AWS CLI**
   ```bash
   curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o awscliv2.zip
   unzip awscliv2.zip && sudo ./aws/install
   aws configure
   # Enter: Access Key ID, Secret Access Key, default region (us-east-1), output (json)
   ```

3. **Set up provider authentication** — Terraform reads credentials from `~/.aws/credentials` automatically after `aws configure`. For CI/CD, use environment variables:
   ```bash
   export AWS_ACCESS_KEY_ID=AKIAIOSFODNN7EXAMPLE
   export AWS_SECRET_ACCESS_KEY=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
   export AWS_DEFAULT_REGION=us-east-1
   ```

4. **Write the provider block** — see the example above.

5. **Run the Terraform workflow**
   ```bash
   terraform init       # downloads hashicorp/aws provider
   terraform validate   # checks HCL syntax
   terraform fmt        # formats files in place
   terraform plan       # shows proposed infrastructure changes
   terraform apply      # creates resources (prompts for confirmation)
   terraform destroy    # tears down all managed resources
   ```

6. **Default state location** — local `terraform.tfstate` file in working directory. In production, switch to the S3 + DynamoDB backend by uncommenting the `backend "s3"` block and re-running `terraform init`.

***

### Terraform Provider Deep-Dive: Azure

#### Full Working Minimal Example

```hcl
# azure_main.tf — Create a Resource Group, VNet, and Linux VM

terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.110"
    }
  }

  # Optional: Remote state in Azure Blob
  # backend "azurerm" {
  #   resource_group_name  = "tf-state-rg"
  #   storage_account_name = "mytfstate123"
  #   container_name       = "tfstate"
  #   key                  = "prod/vm/terraform.tfstate"
  # }
}

variable "location" {
  type    = string
  default = "East US"
}

variable "admin_username" {
  type    = string
  default = "azureuser"
}

variable "admin_password" {
  type      = string
  sensitive = true
}

provider "azurerm" {
  features {}
  # Reads from env vars: ARM_CLIENT_ID, ARM_CLIENT_SECRET,
  # ARM_TENANT_ID, ARM_SUBSCRIPTION_ID
  # Or from `az login` Application Default Credentials
}

resource "azurerm_resource_group" "main" {
  name     = "dev-rg"
  location = var.location
}

resource "azurerm_virtual_network" "main" {
  name                = "dev-vnet"
  address_space       = ["10.0.0.0/16"]
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
}

resource "azurerm_subnet" "internal" {
  name                 = "internal"
  resource_group_name  = azurerm_resource_group.main.name
  virtual_network_name = azurerm_virtual_network.main.name
  address_prefixes     = ["10.0.1.0/24"]
}

resource "azurerm_public_ip" "web" {
  name                = "dev-public-ip"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
  allocation_method   = "Dynamic"
}

resource "azurerm_network_interface" "web" {
  name                = "dev-nic"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name

  ip_configuration {
    name                          = "internal"
    subnet_id                     = azurerm_subnet.internal.id
    private_ip_address_allocation = "Dynamic"
    public_ip_address_id          = azurerm_public_ip.web.id
  }
}

resource "azurerm_linux_virtual_machine" "web" {
  name                            = "dev-vm"
  resource_group_name             = azurerm_resource_group.main.name
  location                        = azurerm_resource_group.main.location
  size                            = "Standard_B1s"
  admin_username                  = var.admin_username
  admin_password                  = var.admin_password
  disable_password_authentication = false

  network_interface_ids = [azurerm_network_interface.web.id]

  os_disk {
    caching              = "ReadWrite"
    storage_account_type = "Standard_LRS"
  }

  source_image_reference {
    publisher = "RedHat"
    offer     = "RHEL"
    sku       = "9-lvm"
    version   = "latest"
  }

  tags = {
    Environment = "dev"
  }
}

output "public_ip_address" {
  value = azurerm_public_ip.web.ip_address
}
```

#### Step-by-Step: Running Terraform on Azure

1. **Install Terraform CLI**
   ```bash
   sudo dnf config-manager --add-repo https://rpm.releases.hashicorp.com/RHEL/hashicorp.repo
   sudo dnf install terraform -y
   terraform version
   ```

2. **Install and configure Azure CLI**
   ```bash
   rpm --import https://packages.microsoft.com/keys/microsoft.asc
   dnf install azure-cli -y
   az login
   az account set --subscription "My Subscription Name"
   az account show
   ```

3. **Set up provider authentication** — `az login` sets Application Default Credentials that `azurerm` reads automatically. For CI/CD, use a Service Principal:
   ```bash
   az ad sp create-for-rbac --name "terraform-sp" --role Contributor \
     --scopes /subscriptions/<subscription-id> --output json
   # Set returned values as environment variables:
   export ARM_CLIENT_ID="<appId>"
   export ARM_CLIENT_SECRET="<password>"
   export ARM_TENANT_ID="<tenant>"
   export ARM_SUBSCRIPTION_ID="<subscriptionId>"
   ```

4. **Write the provider block** — include `features {}` — this is mandatory even if empty.

5. **Run the Terraform workflow**
   ```bash
   terraform init
   terraform validate
   terraform fmt
   terraform plan -var="admin_password=SecurePass123!"
   terraform apply -var="admin_password=SecurePass123!"
   terraform destroy
   ```

6. **Default state location** — local `terraform.tfstate`. For team use, set up an Azure Blob backend by creating a Storage Account and container, then uncomment the `backend "azurerm"` block and run `terraform init -reconfigure`.

***

### Terraform Provider Deep-Dive: GCP

#### Full Working Minimal Example

```hcl
# gcp_main.tf — Create a VPC, subnetwork, and Compute Engine instance

terraform {
  required_providers {
    google = {
      source  = "hashicorp/google"
      version = "~> 5.0"
    }
  }

  # Optional: Remote state in GCS
  # backend "gcs" {
  #   bucket = "my-tf-state-bucket"
  #   prefix = "prod/compute"
  # }
}

variable "project_id" {
  type = string
}

variable "region" {
  type    = string
  default = "us-central1"
}

variable "zone" {
  type    = string
  default = "us-central1-a"
}

provider "google" {
  project = var.project_id
  region  = var.region
  zone    = var.zone
  # Reads from GOOGLE_APPLICATION_CREDENTIALS env var
  # or from `gcloud auth application-default login`
}

resource "google_compute_network" "main" {
  name                    = "dev-vpc"
  auto_create_subnetworks = false
}

resource "google_compute_subnetwork" "public" {
  name          = "dev-subnet"
  ip_cidr_range = "10.0.1.0/24"
  region        = var.region
  network       = google_compute_network.main.id
}

resource "google_compute_firewall" "allow_ssh_http" {
  name    = "allow-ssh-http"
  network = google_compute_network.main.name

  allow {
    protocol = "tcp"
    ports    = ["22", "80"]
  }

  source_ranges = ["0.0.0.0/0"]
  target_tags   = ["web"]
}

resource "google_compute_instance" "web" {
  name         = "dev-web-server"
  machine_type = "e2-micro"
  zone         = var.zone
  tags         = ["web"]

  boot_disk {
    initialize_params {
      image = "rhel-cloud/rhel-9"
      size  = 20
    }
  }

  network_interface {
    subnetwork = google_compute_subnetwork.public.id
    access_config {
      # Ephemeral external IP
    }
  }

  metadata = {
    enable-oslogin = "TRUE"
  }

  labels = {
    environment = "dev"
  }
}

output "instance_external_ip" {
  value = google_compute_instance.web.network_interface[0].access_config[0].nat_ip
}
```

#### Step-by-Step: Running Terraform on GCP

1. **Install Terraform CLI**
   ```bash
   sudo dnf config-manager --add-repo https://rpm.releases.hashicorp.com/RHEL/hashicorp.repo
   sudo dnf install terraform -y
   terraform version
   ```

2. **Install and configure Google Cloud CLI**
   ```bash
   # Download gcloud SDK
   curl -O https://dl.google.com/dl/cloudsdk/channels/rapid/downloads/google-cloud-cli-linux-x86_64.tar.gz
   tar -xf google-cloud-cli-linux-x86_64.tar.gz
   ./google-cloud-sdk/install.sh
   source ~/.bashrc

   # Authenticate
   gcloud auth login
   gcloud auth application-default login   # sets ADC for Terraform
   gcloud config set project my-project-id
   ```

3. **Set up provider authentication** — `gcloud auth application-default login` generates `~/.config/gcloud/application_default_credentials.json` which Terraform reads automatically. For CI/CD, use a Service Account key:
   ```bash
   # Create service account and key
   gcloud iam service-accounts create terraform-sa \
     --display-name="Terraform Service Account"
   gcloud projects add-iam-policy-binding my-project-id \
     --member="serviceAccount:terraform-sa@my-project-id.iam.gserviceaccount.com" \
     --role="roles/editor"
   gcloud iam service-accounts keys create key.json \
     --iam-account=terraform-sa@my-project-id.iam.gserviceaccount.com
   export GOOGLE_APPLICATION_CREDENTIALS="$(pwd)/key.json"
   ```

4. **Enable required APIs** — GCP services must be explicitly enabled before Terraform can create resources:
   ```bash
   gcloud services enable compute.googleapis.com
   gcloud services enable storage.googleapis.com
   gcloud services enable iam.googleapis.com
   ```

5. **Run the Terraform workflow**
   ```bash
   terraform init
   terraform validate
   terraform fmt
   terraform plan -var="project_id=my-project-id"
   terraform apply -var="project_id=my-project-id"
   terraform destroy -var="project_id=my-project-id"
   ```

6. **Default state location** — local `terraform.tfstate`. For team use, create a GCS bucket and uncomment the `backend "gcs"` block, then run `terraform init -migrate-state` to move existing state to the bucket.
