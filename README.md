## SECTION 1

| **Area** | **AWS** | **Azure** | **GCP** |
|---|---|---|---|
| **Definition / Core Philosophy** | **Amazon Web Services (AWS)** — broadest public cloud ecosystem with strong service depth, infrastructure primitives, and global scale. | **Microsoft Azure** — enterprise-first cloud tightly integrated with Microsoft identity, hybrid, and platform services. | **Google Cloud Platform (GCP)** — engineering-focused cloud with strong data, Kubernetes, and global network design. |
| **Compute Services (VMs, serverless, containers)** | **Amazon EC2** — on-demand virtual machines; **AWS Lambda** — event-driven serverless compute; **Amazon ECS / Amazon EKS** — managed container orchestration. | **Azure Virtual Machines** — on-demand Windows/Linux VMs; **Azure Functions** — event-driven serverless code; **Azure Container Apps / AKS** — managed containers and Kubernetes. | **Compute Engine** — on-demand virtual machines; **Cloud Functions / Cloud Run** — serverless execution and containers; **Google Kubernetes Engine (GKE)** — managed Kubernetes. |
| **Storage Services (object, block, file)** | **Amazon S3** — durable object storage; **Amazon EBS** — block storage for EC2; **Amazon EFS** — managed NFS file storage. | **Azure Blob Storage** — massively scalable object storage; **Azure Disk Storage** — high-performance block storage; **Azure Files** — managed SMB/NFS file shares. | **Cloud Storage** — object storage; **Persistent Disk** — block storage for VMs; **Filestore** — managed file storage. |
| **Networking (VPC equivalent, load balancers, DNS)** | **Amazon VPC** — logically isolated virtual network; **Elastic Load Balancing (ALB/NLB)** — distributes traffic across targets; **Amazon Route 53** — managed DNS and routing. | **Azure Virtual Network** — private cloud network boundary; **Azure Load Balancer / Azure Application Gateway** — L4 and L7 traffic distribution; **Azure DNS** — managed DNS hosting. | **Virtual Private Cloud (VPC)** — global private cloud network; **Cloud Load Balancing** — distributed L4/L7 load balancing; **Cloud DNS** — managed authoritative DNS. |
| **IAM & Security (identity, roles, policies, secrets)** | **AWS Identity and Access Management (IAM)** — users, roles, and policy-based access control; **AWS Secrets Manager** — managed secret storage and rotation; **AWS KMS** — managed encryption keys. | **Microsoft Entra ID** — identity and SSO platform; **Azure Role-Based Access Control (RBAC)** — authorization by role assignments; **Azure Key Vault** — managed secrets, keys, and certificates. | **Cloud IAM** — role-based access control for Google Cloud resources; **Secret Manager** — managed secrets storage; **Cloud Key Management Service (Cloud KMS)** — managed encryption keys. |
| **Managed Kubernetes Service** | **Amazon EKS** — managed Kubernetes control plane integrated with AWS networking and IAM. | **Azure Kubernetes Service (AKS)** — managed Kubernetes integrated with Azure networking, identity, and monitoring. | **Google Kubernetes Engine (GKE)** — Google-managed Kubernetes with strong autoscaling and fleet features. |
| **CI/CD Native Tools** | **AWS CodeCommit / CodeBuild / CodeDeploy / CodePipeline** — native source, build, deploy, and pipeline services. | **Azure Repos / Azure Pipelines / Azure Artifacts** — native source control, CI/CD, and package management. | **Cloud Build / Cloud Deploy / Artifact Registry / Source Repositories** — native build, release, and artifact services. |
| **Monitoring & Logging** | **Amazon CloudWatch** — metrics, logs, alarms, and dashboards; **AWS X-Ray** — distributed tracing. | **Azure Monitor** — observability for apps and infrastructure; **Log Analytics** — centralized log querying; **Azure Managed Grafana** — managed dashboards. | **Cloud Monitoring** — metrics and alerting; **Cloud Logging** — centralized logs; **Cloud Trace** — distributed tracing. |
| **Pricing Model Philosophy** | **Pay-as-you-go + Reserved + Savings Plans + Spot** — flexible pricing with strong discounting for predictable usage. | **Pay-as-you-go + Reserved + Spot + Hybrid Benefit** — cost model optimized for Microsoft estate and license reuse. | **Pay-as-you-go + Committed Use Discounts + Spot VMs** — simple sustained/committed discounts with strong analytics economics. |
| **CLI Tool Name** | **AWS CLI (`aws`)** — command-line interface for managing AWS services. | **Azure CLI (`az`)** — command-line interface for managing Azure resources. | **Google Cloud CLI (`gcloud`)** — command-line interface for managing GCP services. |
| **Key Certifications** | **AWS Certified Solutions Architect**, **AWS Certified SysOps Administrator**, **AWS Certified DevOps Engineer - Professional** — core architecting and operations tracks. | **Azure Administrator Associate**, **Azure Solutions Architect Expert**, **DevOps Engineer Expert** — core operations and platform design tracks. | **Associate Cloud Engineer**, **Professional Cloud Architect**, **Professional Cloud DevOps Engineer** — core operations and architecture tracks. |
| **Best Use Case / Industry Fit** | **AWS** — best for service breadth, startups to enterprises, cloud-native platforms, and large-scale infrastructure automation. | **Azure** — best for Microsoft-heavy enterprises, hybrid identity, Windows workloads, regulated industries, and enterprise governance. | **GCP** — best for data platforms, Kubernetes-centric engineering teams, analytics/ML, and globally distributed services. |

### Q&A

**Q:** When do you recommend a true multi-cloud architecture instead of a primary-cloud-with-DR pattern?  
**A:** Use true multi-cloud only when there is a hard business driver: regulatory separation, M&A platform coexistence, extreme resilience requirements, or strategic avoidance of provider lock-in. If the driver is weak, a primary cloud plus cross-region DR is usually cheaper, simpler, and easier to operate.

**Q:** What is the biggest mistake candidates make when designing multi-cloud platforms?  
**A:** They try to make every layer identical across clouds. In production, standardize the control plane patterns, security guardrails, observability model, and delivery workflow, but allow cloud-native implementation under the hood.

**Q:** How do you handle identity consistently across AWS, Azure, and GCP?  
**A:** Use one enterprise identity provider as the source of truth, federate into each cloud, prefer short-lived credentials, and map least-privilege roles per environment. Avoid long-lived static keys unless there is no federated alternative.

**Q:** How do you make Kubernetes portable across clouds without losing cloud-native advantages?  
**A:** Keep app manifests, Helm charts, admission policies, and GitOps flow portable; abstract ingress, storage classes, and secrets carefully; and isolate cloud-specific dependencies behind platform modules. The goal is workload portability, not total infrastructure sameness.

**Q:** What do interviewers want to hear about multi-cloud observability?  
**A:** They want a unified operating model: common SLOs, standard labels/tags, centralized alert routing, and a shared incident process. Cloud-native telemetry is still useful, but incident responders should not need three different mental models during an outage.

## SECTION 2

| **Area** | **RHEL** | **Ubuntu** | **Debian** | **CentOS** |
|---|---|---|---|---|
| **Package Manager (command + example)** | **`dnf`** — RPM package manager for install/update; example: `sudo dnf install nginx -y`. | **`apt`** — DEB package manager for install/update; example: `sudo apt install nginx -y`. | **`apt`** — DEB package manager for stable package management; example: `sudo apt install nginx -y`. | **`dnf`** (or `yum` on older releases) — RPM package manager; example: `sudo dnf install nginx -y`. |
| **Service Management (exact start/stop/enable commands)** | **`systemctl`** — `sudo systemctl start nginx && sudo systemctl stop nginx && sudo systemctl enable nginx`. | **`systemctl`** — `sudo systemctl start nginx && sudo systemctl stop nginx && sudo systemctl enable nginx`. | **`systemctl`** — `sudo systemctl start nginx && sudo systemctl stop nginx && sudo systemctl enable nginx`. | **`systemctl`** — `sudo systemctl start nginx && sudo systemctl stop nginx && sudo systemctl enable nginx`. |
| **Firewall Tool & commands** | **`firewalld` / `firewall-cmd`** — `sudo firewall-cmd --permanent --add-service=http && sudo firewall-cmd --reload`. | **`ufw`** — `sudo ufw allow 80/tcp && sudo ufw enable`. | **`nftables`** (common modern default) or `iptables` — example: `sudo nft add rule inet filter input tcp dport 80 accept`. | **`firewalld` / `firewall-cmd`** — `sudo firewall-cmd --permanent --add-service=http && sudo firewall-cmd --reload`. |
| **Default Shell** | **`bash`** — Bourne Again Shell is the standard admin shell in most enterprise builds. | **`bash`** — common default login shell for server administration. | **`bash`** — default interactive shell for most installations. | **`bash`** — standard admin shell in RHEL-compatible environments. |
| **File System Layout differences** | **`/etc/sysconfig`** often stores distro-specific service and network settings; RPM repos live in **`/etc/yum.repos.d/`**. | **`/etc/netplan/`** is commonly used for network config; APT repos live in **`/etc/apt/`**. | **`/etc/apt/`** is central for package sources; layout is minimal and conservative compared with Ubuntu. | Similar to RHEL, with **`/etc/sysconfig`** and **`/etc/yum.repos.d/`** common in admin workflows. |
| **User & Permission Management key commands** | **`useradd` `usermod` `passwd` `id` `chmod` `chown` `visudo`** — standard Linux identity and permission toolchain. | **`adduser` `usermod` `passwd` `id` `chmod` `chown` `visudo`** — Ubuntu often prefers friendlier `adduser`. | **`adduser` `deluser` `passwd` `id` `chmod` `chown` `visudo`** — common Debian admin set. | **`useradd` `usermod` `passwd` `id` `chmod` `chown` `visudo`** — same operational pattern as RHEL. |
| **Kernel Update approach** | **`dnf update kernel` + reboot** — production teams often pair with maintenance windows or live patching where licensed. | **`apt upgrade` / `apt full-upgrade` + reboot** — Livepatch may reduce reboot frequency in supported estates. | **`apt full-upgrade` + reboot** — conservative but straightforward kernel lifecycle. | **`dnf update kernel` + reboot** — mirrors RHEL operational model. |
| **Enterprise Support availability** | **Red Hat subscription support** — full commercial support, lifecycle management, and compliance ecosystem. | **Canonical support / Ubuntu Pro** — commercial support with security and operations add-ons. | **Community support** — strong ecosystem, but no default single-vendor enterprise contract. | **Varies** — CentOS Stream is community-driven and not equivalent to paid RHEL support. |
| **Best Use Case in DevOps/Cloud** | Enterprise production platforms, regulated environments, OpenShift, hardened middleware, and long-lifecycle servers. | Cloud-native apps, developer-friendly servers, Kubernetes nodes, CI runners, and fast-moving SaaS teams. | Minimal, stable base OS for appliances, lightweight servers, and conservative infrastructure. | Lab environments, RHEL-adjacent testing, rebuild-compatible workflows, and non-production standardization. |
| **SELinux / AppArmor default** | **SELinux** — enabled by default and important for enterprise hardening. | **AppArmor** — commonly enabled by default on Ubuntu. | **AppArmor** — available and commonly used, though profiles may be lighter than Ubuntu defaults. | **SELinux** — typically enabled by default in RHEL-compatible installs. |
| **Common interview gotcha** | Turning off SELinux instead of writing the right context or policy is a red flag. | Assuming Ubuntu networking always uses old `/etc/network/interfaces` instead of checking Netplan or cloud-init is a common miss. | Forgetting that Debian is often leaner and may not ship the same convenience packages as Ubuntu can break automation. | Saying “CentOS equals RHEL” without explaining CentOS Stream versus legacy CentOS Linux shows outdated platform awareness. |

### Cheat Sheet

| **Task** | **RHEL** | **Ubuntu** | **Debian** | **CentOS** |
|---|---|---|---|---|
| **1. Update package index / metadata** | `sudo dnf makecache` | `sudo apt update` | `sudo apt update` | `sudo dnf makecache` |
| **2. Upgrade installed packages** | `sudo dnf upgrade -y` | `sudo apt upgrade -y` | `sudo apt upgrade -y` | `sudo dnf upgrade -y` |
| **3. Install NGINX** | `sudo dnf install nginx -y` | `sudo apt install nginx -y` | `sudo apt install nginx -y` | `sudo dnf install nginx -y` |
| **4. Search package** | `dnf search nginx` | `apt search nginx` | `apt search nginx` | `dnf search nginx` |
| **5. Start service** | `sudo systemctl start nginx` | `sudo systemctl start nginx` | `sudo systemctl start nginx` | `sudo systemctl start nginx` |
| **6. Enable at boot** | `sudo systemctl enable nginx` | `sudo systemctl enable nginx` | `sudo systemctl enable nginx` | `sudo systemctl enable nginx` |
| **7. Open HTTP in firewall** | `sudo firewall-cmd --permanent --add-service=http && sudo firewall-cmd --reload` | `sudo ufw allow 80/tcp && sudo ufw reload` | `sudo nft add rule inet filter input tcp dport 80 accept` | `sudo firewall-cmd --permanent --add-service=http && sudo firewall-cmd --reload` |
| **8. Show IP address** | `ip addr show` | `ip addr show` | `ip addr show` | `ip addr show` |
| **9. View recent service logs** | `sudo journalctl -u nginx -n 50 --no-pager` | `sudo journalctl -u nginx -n 50 --no-pager` | `sudo journalctl -u nginx -n 50 --no-pager` | `sudo journalctl -u nginx -n 50 --no-pager` |
| **10. Create admin user** | `sudo useradd -m devops && sudo passwd devops && sudo usermod -aG wheel devops` | `sudo adduser devops && sudo usermod -aG sudo devops` | `sudo adduser devops && sudo usermod -aG sudo devops` | `sudo useradd -m devops && sudo passwd devops && sudo usermod -aG wheel devops` |

### Quick Notes

```bash
# RHEL / CentOS: SELinux checks
getenforce
sestatus
sudo restorecon -Rv /var/www/html

# Ubuntu / Debian: AppArmor checks
sudo aa-status
sudo systemctl status apparmor

# Cross-distro: service + networking triage
systemctl status nginx
ss -tulpn
curl -I http://localhost
journalctl -xe
```
