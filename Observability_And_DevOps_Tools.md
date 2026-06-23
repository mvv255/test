## SECTION 3 — Observability & Monitoring Tools Comparison Table

| **Area** | **Datadog** | **Dynatrace** | **Grafana** | **Prometheus** | **New Relic** | **ELK Stack (Elasticsearch / Logstash / Kibana)** | **AWS CloudWatch** |
|---|---|---|---|---|---|---|---|
| **Type** | All-in-one observability platform: APM, metrics, logs, traces, RUM, security | All-in-one observability platform with AI-assisted root cause analysis | Visualization and dashboard platform; often paired with metrics/logs/traces backends | Metrics-first monitoring and alerting system | Full-stack observability: APM, logs, metrics, traces | Log analytics and search stack, often extended for observability | AWS-native monitoring, metrics, logs, alarms, dashboards, traces |
| **Data Collection Method** | Agents, integrations, OpenTelemetry, APIs, cloud connectors | OneAgent auto-instrumentation, APIs, cloud integrations | Pulls from multiple backends; Grafana Agent/Alloy can collect and forward | Pull-based scraping of `/metrics`; exporters for infra/apps | Language agents, infrastructure agent, OpenTelemetry, APIs | Beats, Logstash pipelines, APIs, agents | Native AWS service integrations, CloudWatch Agent, API, embedded metrics |
| **Storage Backend** | Proprietary Datadog SaaS backend | Proprietary Dynatrace backend / Grail storage | Usually external backend: Prometheus, Loki, Tempo, Mimir, Elasticsearch, InfluxDB | Built-in TSDB (time-series database) | Proprietary NRDB backend | Elasticsearch stores indexed documents and logs | Managed AWS backend for metrics/logs/events |
| **Visualization / Dashboarding** | Rich dashboards, service maps, infra views, notebooks | Smart dashboards, topology maps, AI-assisted views | Best-in-class customizable dashboards and panels | Basic built-in expression/UI; usually paired with Grafana | New Relic One dashboards and NRQL-based widgets | Kibana dashboards, discovery, search, Lens visualizations | CloudWatch Dashboards and service-native views |
| **Alerting Mechanism** | Threshold, anomaly, composite, forecast alerts; integrations with PagerDuty/Slack | AI-driven alerting, anomaly detection, problem correlation | Grafana Alerting, contact points, policies | Alertmanager handles routing, grouping, silencing | NRQL-based alert conditions and AI correlations | Kibana alerting, Watcher, connectors | CloudWatch Alarms, anomaly detection, SNS/EventBridge actions |
| **Pricing Model (open-source vs SaaS)** | Commercial SaaS, priced by host/container/ingest/features | Commercial SaaS, enterprise pricing | Open-source core; Grafana Cloud paid SaaS | Open-source, self-hosted | Commercial SaaS | Open-source self-hosted or Elastic Cloud paid service | Pay-as-you-go AWS managed service |
| **Best For (use case)** | Enterprises needing broad observability with fast onboarding | Large enterprises needing deep topology and automated RCA | Unified dashboarding across many tools and clouds | Kubernetes/cloud-native metrics and SRE-style alerting | App-centric full-stack monitoring for engineering teams | Centralized log aggregation, search, compliance, SIEM-style analysis | AWS-first environments needing native monitoring with low setup overhead |
| **Integration with Kubernetes** | Strong: Helm, Operator, APM injection, cluster explorer | Strong: OneAgent Operator, auto-discovery, topology mapping | Strong when paired with Prometheus/Loki/Tempo via Helm/Operator | Excellent: standard choice for K8s metrics via Prometheus Operator | Strong: Helm-based integrations and Kubernetes explorer | Good: Beats/Elastic Agent/Operator on Kubernetes | Strong for EKS with Container Insights and CloudWatch Agent |
| **Key Interview Question about this tool** | How do you control Datadog cost when cardinality explodes? | What makes Dynatrace different from traditional monitoring agents? | How is Grafana different from Prometheus? | Why does Prometheus use pull-based scraping instead of push? | How does New Relic instrument JVM or Node.js apps? | When would you use Filebeat only vs Filebeat + Logstash? | When should you use CloudWatch alone vs exporting to a third-party platform? |

### The Four Pillars of Observability

**Metrics** are numeric time-series signals such as CPU usage, latency, request count, and error rate. They are the fastest way to detect that something is wrong because they are lightweight, aggregatable, and ideal for dashboards, SLOs, and alert thresholds. [web:6][web:7]

**Logs** are timestamped records of discrete events produced by applications, systems, and network devices. They provide detailed diagnostic context and are usually the best signal for understanding why a failure happened after metrics reveal that a problem exists. [web:6][web:7]

**Traces** follow a single request as it moves across distributed services and record spans for each hop. They are essential in microservices because they expose latency bottlenecks, retries, fan-out patterns, and downstream dependencies that metrics alone cannot explain. [web:5][web:7]

**Events** are meaningful state changes such as deployments, autoscaling actions, restarts, failovers, and configuration changes. They create the correlation layer in observability because they help teams connect a symptom spike to the exact operational change that likely caused it. [web:6][web:7]

---

## SECTION 4 — DevOps Tools Deep-Dive

### Docker

#### 1. What it is

Docker is a containerization platform that packages an application and its runtime dependencies into a portable image. The core idea is OS-level isolation using Linux namespaces and cgroups so the same image can run consistently across laptops, VMs, and cloud hosts.

#### 2. How it works

Key components:
- **Docker Engine (`dockerd`)** — daemon that manages images, containers, networks, and volumes.
- **Docker CLI (`docker`)** — client used to build, run, inspect, and manage containers.
- **Dockerfile** — declarative build recipe for an image.
- **Registry** — remote image store such as Docker Hub, ECR, ACR, or GCR.
- **Image** — immutable layered artifact.
- **Container** — running instance of an image with a writable layer.

Typical workflow:
1. Write a `Dockerfile`.
2. Build an image with `docker build`.
3. Push the image to a registry.
4. Pull and run it on a target host or orchestrator.
5. Mount volumes for persistence and attach to networks for connectivity.

Production concepts:
- Layer caching speeds builds.
- Multi-stage builds reduce final image size.
- Containers should run as non-root.
- Secrets should never be baked into images.
- Volumes persist data outside container lifecycle.

#### 3. Key commands / YAML examples

```dockerfile
FROM maven:3.9-eclipse-temurin-21 AS build
WORKDIR /app
COPY pom.xml .
RUN mvn dependency:go-offline -B
COPY src ./src
RUN mvn package -DskipTests

FROM eclipse-temurin:21-jre-alpine
WORKDIR /app
COPY --from=build /app/target/app.jar app.jar
RUN addgroup -S app && adduser -S app -G app
USER app
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

```bash
docker build -t myapp:1.0 .
docker run -d --name myapp -p 8080:8080 myapp:1.0
docker ps
docker logs -f myapp
docker exec -it myapp /bin/sh
docker stop myapp
docker rm myapp
docker volume create appdata
docker network create app-net
docker tag myapp:1.0 <account>.dkr.ecr.us-east-1.amazonaws.com/myapp:1.0
docker push <account>.dkr.ecr.us-east-1.amazonaws.com/myapp:1.0
```

#### 4. Interview must-know topics

- Image vs container.
- Dockerfile layer caching strategy.
- `ENTRYPOINT` vs `CMD`.
- Bind mounts vs volumes.
- Bridge vs host networking.
- Multi-stage builds.
- Rootless and non-root containers.
- Registry authentication and image scanning.
- Copy-on-write filesystem behavior.
- Why containers are not VMs.

#### 5. Top 5 Interview Q&A

**Q:** What is the difference between an image and a container?  
**A:** An image is an immutable template made of layers, while a container is a running instance of that image with a writable runtime layer.

**Q:** Why use multi-stage builds?  
**A:** They separate build dependencies from runtime artifacts, which reduces image size, startup time, and attack surface.

**Q:** How do Docker volumes help in production?  
**A:** They keep state outside the container lifecycle, so data survives container recreation and can be backed up independently.

**Q:** What is the purpose of `ENTRYPOINT` and `CMD`?  
**A:** `ENTRYPOINT` defines the executable, and `CMD` provides default arguments that can be overridden.

**Q:** Why is running containers as root discouraged?  
**A:** A compromised root container increases blast radius; production images should use a dedicated non-root user and least privilege.

---

### Kubernetes

#### 1. What it is

Kubernetes is a container orchestration platform that automates deployment, scaling, service discovery, and self-healing for containerized applications. Its central model is desired-state reconciliation.

#### 2. How it works

Control plane:
- **API Server** — cluster entry point.
- **etcd** — source of truth for cluster state.
- **Scheduler** — places pods on nodes.
- **Controller Manager** — enforces desired state.
- **Cloud Controller Manager** — integrates with cloud APIs.

Node components:
- **kubelet** — node agent.
- **kube-proxy** — service networking rules.
- **container runtime** — typically containerd or CRI-O.

Core objects:
- **Pod** — smallest deployable unit.
- **Deployment** — manages stateless replica updates.
- **Service** — stable virtual endpoint.
- **Ingress** — L7 routing to services.
- **ConfigMap / Secret** — externalized config.
- **Namespace** — logical isolation boundary.
- **RBAC** — authorization model.
- **HPA** — horizontal scaling by metrics.

#### 3. Key commands / YAML examples

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
  namespace: production
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: myregistry/myapp:1.0
        ports:
        - containerPort: 8080
        resources:
          requests:
            cpu: "250m"
            memory: "256Mi"
          limits:
            cpu: "500m"
            memory: "512Mi"
        envFrom:
        - configMapRef:
            name: myapp-config
```

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-svc
  namespace: production
spec:
  selector:
    app: myapp
  ports:
  - port: 80
    targetPort: 8080
  type: ClusterIP
```

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: myapp-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: myapp
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```

```bash
kubectl get pods -A
kubectl describe pod myapp-123 -n production
kubectl logs -f deploy/myapp -n production
kubectl exec -it pod/myapp-123 -n production -- /bin/sh
kubectl rollout status deploy/myapp -n production
kubectl rollout undo deploy/myapp -n production
kubectl get svc,ingress -n production
kubectl top pods -n production
```

#### 4. Interview must-know topics

- Pod vs Deployment vs StatefulSet.
- ClusterIP vs NodePort vs LoadBalancer.
- Ingress controller role.
- Readiness vs liveness vs startup probes.
- Requests vs limits and OOMKill behavior.
- ConfigMaps vs Secrets.
- Namespaces and multi-tenant isolation.
- RBAC roles, rolebindings, service accounts.
- HPA vs Cluster Autoscaler.
- Taints, tolerations, affinity, anti-affinity.

#### 5. Top 5 Interview Q&A

**Q:** What is the difference between a Pod and a Deployment?  
**A:** A Pod is the basic runtime unit; a Deployment manages replica count, rolling updates, and rollback for pods.

**Q:** Why do services matter in Kubernetes?  
**A:** Pods are ephemeral, so Services provide stable DNS and virtual IPs for internal or external connectivity.

**Q:** What happens if readiness probes fail?  
**A:** The pod stays running but is removed from service endpoints, so traffic is not sent to it.

**Q:** How does HPA work?  
**A:** HPA reads metrics such as CPU or custom metrics and changes replica count to match a target utilization.

**Q:** Why is RBAC important?  
**A:** It enforces least privilege so users, service accounts, and apps get only the Kubernetes permissions they need.

---

### Ansible

#### 1. What it is

Ansible is an agentless automation tool for configuration management, provisioning, patching, and orchestration. It uses YAML playbooks and usually connects over SSH.

#### 2. How it works

Core pieces:
- **Inventory** — hosts and groups.
- **Playbook** — ordered automation definition.
- **Play** — maps tasks to a host group.
- **Task** — single module execution.
- **Module** — reusable action unit such as `yum`, `service`, `copy`, `template`.
- **Role** — structured reusable automation bundle.
- **Vault** — secret encryption.
- **Handlers** — event-driven tasks triggered by changes.

Execution flow:
1. Parse inventory.
2. Evaluate variables and facts.
3. Connect to target hosts.
4. Run tasks in order.
5. Trigger handlers only when notified.
6. Report changed/ok/failed state.

#### 3. Key commands / YAML examples

```bash
ansible all -i inventory.ini -m ping
ansible web -i inventory.ini -m shell -a "uptime"
ansible-playbook -i inventory.ini site.yml
ansible-playbook -i inventory.ini site.yml --check
ansible-playbook -i inventory.ini site.yml --diff
ansible-vault encrypt group_vars/all/vault.yml
```

```yaml
- name: Configure web tier
  hosts: web
  become: true
  tasks:
    - name: Install nginx
      ansible.builtin.dnf:
        name: nginx
        state: present

    - name: Deploy config
      ansible.builtin.template:
        src: nginx.conf.j2
        dest: /etc/nginx/nginx.conf
      notify: Restart nginx

    - name: Ensure service enabled
      ansible.builtin.service:
        name: nginx
        state: started
        enabled: true

  handlers:
    - name: Restart nginx
      ansible.builtin.service:
        name: nginx
        state: restarted
```

#### 4. Interview must-know topics

- Idempotency.
- Facts and variable precedence.
- Inventory structure and grouping.
- Roles and reusable playbook design.
- Templates with Jinja2.
- Handlers and change-driven restarts.
- Vault for secrets.
- `--check` dry run behavior.
- Ad-hoc commands vs playbooks.
- Dynamic inventory for AWS/Azure/GCP.

#### 5. Top 5 Interview Q&A

**Q:** What does agentless mean in Ansible?  
**A:** Managed nodes do not need a long-running Ansible agent; Ansible connects over SSH or WinRM and executes modules remotely.

**Q:** Why is idempotency important?  
**A:** It ensures repeated runs converge on the same desired state without causing drift or duplicate actions.

**Q:** When would you use a role?  
**A:** Use roles to organize reusable automation for services or patterns like `nginx`, `common`, or `hardening`.

**Q:** What is the difference between `copy` and `template`?  
**A:** `copy` transfers a static file; `template` renders Jinja2 variables before placing the file on the target.

**Q:** How do you protect secrets in playbooks?  
**A:** Store them in encrypted Vault files and inject the vault password securely through CI/CD or a secret manager.

---

### Maven

#### 1. What it is

Maven is a Java build and dependency management tool centered around the `pom.xml`. It standardizes project structure and the software build lifecycle.

#### 2. How it works

Key concepts:
- **POM** defines artifact metadata, dependencies, plugins, and build settings.
- **Lifecycle** includes phases like `compile`, `test`, `package`, `install`, `deploy`.
- **Plugins** execute goals during lifecycle phases.
- **Local repository** caches dependencies under `~/.m2/repository`.
- **Remote repositories** include Maven Central, Nexus, or Artifactory.
- **Transitive dependency resolution** automatically pulls dependent libraries.

#### 3. Key commands / YAML examples

```bash
mvn clean
mvn compile
mvn test
mvn package
mvn install
mvn deploy
mvn dependency:tree
mvn help:effective-pom
```

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.example</groupId>
  <artifactId>myapp</artifactId>
  <version>1.0.0</version>

  <properties>
    <java.version>21</java.version>
  </properties>

  <dependencies>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
      <version>3.2.5</version>
    </dependency>
  </dependencies>

  <build>
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-compiler-plugin</artifactId>
        <version>3.11.0</version>
        <configuration>
          <source>21</source>
          <target>21</target>
        </configuration>
      </plugin>
    </plugins>
  </build>
</project>
```

#### 4. Interview must-know topics

- Standard directory layout.
- Lifecycle phases and what each does.
- Dependency scopes: compile, provided, runtime, test.
- SNAPSHOT vs release versions.
- Plugin execution.
- Dependency conflicts and mediation.
- Parent POM and multi-module builds.
- Nexus/Artifactory integration.
- Effective POM and inheritance.
- `mvn install` vs `mvn deploy`.

#### 5. Top 5 Interview Q&A

**Q:** What is the purpose of `pom.xml`?  
**A:** It is the project definition file that controls dependencies, plugins, coordinates, repositories, and build configuration.

**Q:** What is the difference between `install` and `deploy`?  
**A:** `install` publishes the artifact to the local Maven repo; `deploy` publishes it to a remote shared repository.

**Q:** Why are dependency scopes important?  
**A:** They control where a dependency is available, such as compile time, runtime, or tests only, which affects classpath size and packaging.

**Q:** What is a SNAPSHOT version?  
**A:** It is a mutable development version intended for ongoing work, unlike immutable release versions.

**Q:** Why use Nexus or Artifactory?  
**A:** They provide controlled artifact storage, caching, access control, and promotion across environments.

---

### OpenShift

#### 1. What it is

OpenShift is Red Hat’s enterprise Kubernetes platform with built-in security defaults, routing, image registry, operators, and developer tooling. It is more opinionated than upstream Kubernetes.

#### 2. How it works

Major differences from vanilla Kubernetes:
- **Projects** are OpenShift’s managed form of namespaces.
- **Routes** provide native external HTTP/S exposure.
- **Security Context Constraints (SCCs)** enforce pod security rules.
- **Integrated registry** and build workflows are included.
- **`oc` CLI** extends `kubectl`.
- **Operators** are central to platform lifecycle management.
- **CRI-O** is commonly used as the runtime.

#### 3. Key commands / YAML examples

```bash
oc login https://api.cluster.example.com:6443 --token=<token>
oc new-project dev
oc project dev
oc get pods
oc get routes
oc new-app nginx
oc expose svc/nginx
oc describe route nginx
oc adm policy add-scc-to-user anyuid -z default -n dev
```

```yaml
apiVersion: route.openshift.io/v1
kind: Route
metadata:
  name: myapp
  namespace: production
spec:
  to:
    kind: Service
    name: myapp-svc
  port:
    targetPort: 8080
  tls:
    termination: edge
```

#### 4. Interview must-know topics

- OpenShift vs Kubernetes.
- Route vs Ingress.
- Project vs Namespace.
- SCC and non-root container requirements.
- `oc` CLI basics.
- Integrated registry and image streams.
- Operators and OperatorHub.
- BuildConfig / S2I concepts.
- Why some Docker images fail on OpenShift.
- Enterprise governance benefits.

#### 5. Top 5 Interview Q&A

**Q:** What is the biggest security difference between OpenShift and vanilla Kubernetes?  
**A:** OpenShift enforces stricter defaults through SCCs, especially around non-root execution and privilege restrictions.

**Q:** How is a Route different from an Ingress?  
**A:** A Route is an OpenShift-native external routing resource, while Ingress is the upstream Kubernetes abstraction.

**Q:** Why do some containers fail on OpenShift but work on Docker or EKS?  
**A:** Many images expect root privileges or fixed UIDs, but OpenShift’s default SCCs block that behavior.

**Q:** What is `oc`?  
**A:** It is the OpenShift CLI and includes Kubernetes commands plus OpenShift-specific functionality.

**Q:** What are projects in OpenShift?  
**A:** Projects are namespace-based isolation units with added governance, metadata, and access patterns.

---

### Terraform

#### 1. What it is

Terraform is an Infrastructure as Code tool that uses declarative configuration to provision and manage infrastructure. Its most important operational concept is the state file.

#### 2. How it works

Key concepts:
- **Provider** — cloud/API plugin.
- **Resource** — managed infrastructure object.
- **Data source** — read-only lookup of existing data.
- **State** — snapshot of managed resources.
- **Plan** — proposed diff.
- **Apply** — execution of the diff.
- **Module** — reusable infrastructure package.
- **Backend** — where state is stored.
- **Workspace** — separate named states in one config.

Typical production workflow:
1. `terraform init`
2. `terraform validate`
3. `terraform fmt`
4. `terraform plan`
5. Review output
6. `terraform apply`

#### 3. Key commands / YAML examples

```bash
terraform init
terraform fmt
terraform validate
terraform plan
terraform plan -out=tfplan
terraform apply tfplan
terraform destroy
terraform state list
terraform output
terraform workspace list
```

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }

  backend "s3" {
    bucket         = "my-tf-state"
    key            = "prod/network/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "tf-locks"
    encrypt        = true
  }
}

provider "aws" {
  region = "us-east-1"
}

resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
  tags = {
    Name = "prod-vpc"
  }
}
```

```hcl
module "web" {
  source = "./modules/ec2"
  instance_type = "t3.medium"
  subnet_id     = "subnet-123456"
}
```

#### 4. Interview must-know topics

- State file importance and sensitivity.
- Remote backend and locking.
- `plan` vs `apply`.
- Drift detection.
- `count` vs `for_each`.
- Modules and reusability.
- Importing existing resources.
- Workspace limitations.
- Secrets in state.
- Lifecycle meta-arguments like `prevent_destroy`.

#### 5. Top 5 Interview Q&A

**Q:** Why should Terraform state not stay local in a team environment?  
**A:** Local state breaks collaboration, prevents locking, and increases the risk of drift and state corruption.

**Q:** What is the purpose of DynamoDB with an S3 backend?  
**A:** It provides state locking so two applies do not modify the same state simultaneously.

**Q:** What is drift in Terraform?  
**A:** Drift occurs when real infrastructure changes outside Terraform and no longer matches the recorded state or code.

**Q:** When do you prefer `for_each` over `count`?  
**A:** Use `for_each` when resources need stable keys because removing one item does not force index reshuffling.

**Q:** Can Terraform manage existing infrastructure?  
**A:** Yes, by writing the resource block and importing the existing object into state.

---

### Helm

#### 1. What it is

Helm is the package manager for Kubernetes. It packages Kubernetes manifests into versioned charts with configurable values and release history.

#### 2. How it works

Core pieces:
- **Chart** — package containing templates and metadata.
- **`values.yaml`** — default configuration values.
- **Templates** — Go-template Kubernetes manifests.
- **Release** — installed instance of a chart.
- **Repository** — chart distribution endpoint.
- **Upgrade / rollback** — release revision management.

Helm renders templates locally and then applies the resulting manifests to the cluster.

#### 3. Key commands / YAML examples

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
helm search repo nginx
helm install myapp ./mychart -n production --create-namespace
helm upgrade myapp ./mychart -n production -f values-prod.yaml
helm rollback myapp 1 -n production
helm history myapp -n production
helm uninstall myapp -n production
helm lint ./mychart
helm template myapp ./mychart
```

```yaml
apiVersion: v2
name: myapp
description: Sample app chart
type: application
version: 1.0.0
appVersion: "2.0.0"
```

```yaml
replicaCount: 2

image:
  repository: myregistry/myapp
  tag: "1.0.0"

service:
  type: ClusterIP
  port: 80
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "myapp.fullname" . }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: {{ include "myapp.name" . }}
  template:
    metadata:
      labels:
        app: {{ include "myapp.name" . }}
    spec:
      containers:
      - name: myapp
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
```

#### 4. Interview must-know topics

- Chart structure.
- Release and revision concepts.
- `values.yaml` and override hierarchy.
- `helm install` vs `helm upgrade`.
- Rollback behavior.
- Chart templating functions.
- Helm hooks.
- Dependency/subchart management.
- OCI registry support.
- Differences between raw manifests, Kustomize, and Helm.

#### 5. Top 5 Interview Q&A

**Q:** What problem does Helm solve?  
**A:** It standardizes packaging, parameterization, deployment, and rollback of Kubernetes applications.

**Q:** What is a release in Helm?  
**A:** A release is a named installed instance of a chart in a specific namespace.

**Q:** Why use `helm upgrade --atomic`?  
**A:** It rolls back automatically if the upgrade fails, which is safer for production deployments.

**Q:** How are values overridden in Helm?  
**A:** By `values.yaml`, additional `-f` files, and `--set`, with later inputs taking precedence.

**Q:** When is Helm better than raw YAML?  
**A:** Helm is better when the same app needs reusable packaging, environment-specific values, and upgrade history.
