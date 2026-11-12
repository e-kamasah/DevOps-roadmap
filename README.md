# Kubernetes: A Practical Learning Reference

A topic-by-topic reference guide based on the Kubernetes and DevOps
learning sessions.

**Version:** 1.0\
**Status:** Prometheus and Grafana in progress

------------------------------------------------------------------------

## Table of Contents

-   [Part I -- Kubernetes
    Fundamentals](#part-i--kubernetes-fundamentals)
    -   [1. Introduction to Kubernetes](#1-introduction-to-kubernetes)
    -   [2. Kubernetes Architecture](#2-kubernetes-architecture)
    -   [3. Pods and Workload
        Management](#3-pods-and-workload-management)
    -   [4. Namespaces, Labels and
        Selectors](#4-namespaces-labels-and-selectors)
-   [Part II -- Application Deployment and
    Networking](#part-ii--application-deployment-and-networking)
    -   [5. Kubernetes Services](#5-kubernetes-services)
    -   [6. Kubernetes Networking and
        Ingress](#6-kubernetes-networking-and-ingress)
    -   [7. ConfigMaps and Secrets](#7-configmaps-and-secrets)
    -   [8. Kubernetes Storage](#8-kubernetes-storage)
-   [Part III -- Workload Management and
    Security](#part-iii--workload-management-and-security)
    -   [9. Workload Controllers](#9-workload-controllers)
    -   [10. Health Checks and Resource
        Management](#10-health-checks-and-resource-management)
    -   [11. Kubernetes Security](#11-kubernetes-security)
    -   [12. Scheduling and Placement](#12-scheduling-and-placement)
-   [Part IV -- Application and Cluster
    Operations](#part-iv--application-and-cluster-operations)
    -   [13. Helm](#13-helm)
    -   [14. Autoscaling](#14-autoscaling)
    -   [15. Amazon EKS](#15-amazon-eks)
-   [Part V -- Automation and
    Observability](#part-v--automation-and-observability)
    -   [16. GitOps](#16-gitops)
    -   [17. Argo CD](#17-argo-cd)
    -   [18. Monitoring and
        Observability](#18-monitoring-and-observability)
    -   [19. Prometheus and PromQL](#19-prometheus-and-promql)
    -   [20. Grafana](#20-grafana)
-   [Part VI -- Planned Topics](#part-vi--planned-topics)
-   [kubectl Quick Reference](#kubectl-quick-reference)
-   [Helm Quick Reference](#helm-quick-reference)
-   [Important Concepts](#important-concepts)
-   [Learning Status](#learning-status)

------------------------------------------------------------------------

# Part I -- Kubernetes Fundamentals

## 1. Introduction to Kubernetes

### What is Kubernetes?

Kubernetes is an open-source container orchestration platform used to
deploy, manage, scale and maintain containerized applications.

It automates tasks such as:

-   Deploying applications.
-   Restarting failed containers.
-   Scaling application instances.
-   Distributing workloads across machines.
-   Managing application configuration.
-   Exposing applications through networking.

### Container orchestration

Container orchestration is the automated management of containers. For
example, Kubernetes can maintain a desired number of application
instances and replace failed instances.

### Kubernetes cluster

A Kubernetes cluster is a collection of machines that work together to
run containerized applications. It consists of:

-   **Control plane:** manages the cluster.
-   **Worker nodes:** provide the environment where workloads run.

### Declarative configuration

Kubernetes uses a declarative approach: describe the desired outcome
rather than every action needed to achieve it.

``` yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 3
```

This declares that the Deployment should maintain three replicas.

**Key takeaway:** You describe what you want; Kubernetes works to make
the actual state match it.

## 2. Kubernetes Architecture

### Control plane

The control plane manages the cluster.

  -----------------------------------------------------------------------
  Component                           Responsibility
  ----------------------------------- -----------------------------------
  API Server                          Entry point for Kubernetes API
                                      requests

  etcd                                Stores cluster state and
                                      configuration

  Scheduler                           Selects suitable nodes for Pods

  Controller Manager                  Runs controllers that reconcile
                                      desired and actual state
  -----------------------------------------------------------------------

### Worker node

  -----------------------------------------------------------------------
  Component                           Responsibility
  ----------------------------------- -----------------------------------
  Kubelet                             Ensures assigned Pods are running
                                      as expected

  Kube-proxy                          Implements Service networking
                                      behavior, traditionally using
                                      network rules

  Container runtime                   Runs containers, for example
                                      containerd or CRI-O
  -----------------------------------------------------------------------

### Typical request flow

1.  A user submits a Kubernetes configuration.
2.  The API Server receives and validates the request.
3.  The desired state is stored in etcd.
4.  Controllers create or update the required resources.
5.  The Scheduler selects suitable nodes.
6.  Kubelet works with the container runtime to run containers.
7.  Controllers continue reconciling the cluster state.

**Key takeaway:** The control plane manages the cluster; worker nodes
run application workloads.

## 3. Pods and Workload Management

### Pod

A Pod is the smallest deployable unit in Kubernetes. It can contain one
or more containers that share resources such as network identity and
mounted storage. Most commonly, a Pod contains one main application
container.

### ReplicaSet

A ReplicaSet maintains a specified number of matching Pod replicas.

### Deployment

A Deployment manages stateless application workloads and provides:

-   Declarative updates.
-   Scaling.
-   Rolling updates.
-   Rollbacks.
-   ReplicaSet management.

``` bash
kubectl create deployment nginx --image=nginx
kubectl scale deployment nginx --replicas=3
kubectl get deployments
```

### Rolling updates and rollbacks

A rolling update gradually replaces old Pods with new ones. A rollback
returns a Deployment to an earlier revision.

``` bash
kubectl rollout status deployment/nginx
kubectl rollout history deployment/nginx
kubectl rollout undo deployment/nginx
```

### Deployment strategies

-   **RollingUpdate:** gradually replaces existing Pods.
-   **Recreate:** terminates existing Pods before creating replacement
    Pods.

**Key takeaway:** Deployments are the usual way to manage stateless
applications.

## 4. Namespaces, Labels and Selectors

### Namespaces

Namespaces logically separate and organize resources within a cluster.
Common examples include `development`, `staging`, `production` and
`monitoring`.

``` bash
kubectl get namespaces
kubectl create namespace development
kubectl get pods -n development
```

### Labels

Labels are key-value pairs attached to Kubernetes resources.

``` yaml
metadata:
  labels:
    app: frontend
    environment: development
```

### Selectors

Selectors identify resources by matching their labels.

``` yaml
selector:
  matchLabels:
    app: frontend
```

**Key takeaway:** Namespaces organize resources; labels identify them;
selectors find matching resources.

------------------------------------------------------------------------

# Part II -- Application Deployment and Networking

## 5. Kubernetes Services

### What is a Service?

A Service provides a stable network endpoint for accessing a group of
Pods. This is useful because Pods can be replaced and their IP addresses
can change.

### Service types

  -----------------------------------------------------------------------
  Type                                Purpose
  ----------------------------------- -----------------------------------
  ClusterIP                           Exposes a Service internally within
                                      the cluster

  NodePort                            Exposes a Service through a port on
                                      each node

  LoadBalancer                        Exposes a Service through an
                                      external load-balancing mechanism
                                      when supported
  -----------------------------------------------------------------------

### Important Service fields

-   `port`: port exposed by the Service.
-   `targetPort`: port on the destination Pod.
-   `selector`: identifies the Pods that receive traffic.

### Endpoints and EndpointSlices

Endpoints represent the network addresses selected by a Service.
EndpointSlices provide a scalable way to represent those endpoints.

### Service discovery

Kubernetes provides DNS-based service discovery. Applications can use
Service names rather than tracking changing Pod IP addresses.

Example:

``` text
backend-service.default.svc.cluster.local
```

**Key takeaway:** Services provide stable access to Pods and simplify
application-to-application communication.

## 6. Kubernetes Networking and Ingress

### Kubernetes networking

Kubernetes networking supports communication between:

-   Containers within a Pod.
-   Different Pods.
-   Pods and Services.
-   External clients and cluster workloads.

### Network Policies

Network Policies define permitted ingress and egress communication
involving selected Pods. Enforcement requires a network plugin that
supports NetworkPolicy.

### Ingress

Ingress defines rules for managing external HTTP and HTTPS access to
Services. For example, different hostnames can route to different
Services:

``` text
shop.example.com
api.example.com
```

### Ingress Controller

An Ingress resource defines routing rules; an Ingress Controller
implements them.

Examples include Traefik and NGINX Ingress Controller.

### Traefik

Traefik is a reverse proxy and Ingress Controller. Topics covered
included:

-   Installing Traefik with Helm.
-   IngressClass.
-   Ingress resources.
-   Routing requests to Services.
-   Middleware.
-   TLS and HTTPS.

**Key takeaway:** Ingress defines HTTP/HTTPS routing rules; the Ingress
Controller makes those rules work.

## 7. ConfigMaps and Secrets

### ConfigMaps

ConfigMaps store non-sensitive configuration, such as application
settings, database hostnames and configuration files.

### Secrets

Secrets store sensitive configuration, such as passwords, API
credentials and tokens.

Kubernetes Secrets are not automatically secure merely because they are
called Secrets. Production environments need suitable encryption, access
controls and secret-management practices.

### How applications consume configuration

Applications can receive configuration through:

-   Environment variables.
-   Mounted files.
-   References in Pod specifications.

  ConfigMap                     Secret
  ----------------------------- -------------------------------------
  Non-sensitive configuration   Sensitive configuration
  Application settings          Credentials and confidential values

**Key takeaway:** Separate configuration from application images and
apply additional protection to sensitive values.

## 8. Kubernetes Storage

### Volumes

A Volume provides storage to containers in a Pod. Some volumes are
temporary; others use persistent storage.

### PersistentVolume (PV)

A PersistentVolume represents storage made available to a Kubernetes
cluster.

### PersistentVolumeClaim (PVC)

A PersistentVolumeClaim requests storage.

### StorageClass

A StorageClass describes how storage can be dynamically provisioned.

### Dynamic provisioning

Dynamic provisioning allows storage to be created automatically when a
suitable PVC is created.

### Access modes

Common access modes include:

-   `ReadWriteOnce` (RWO).
-   `ReadOnlyMany` (ROX).
-   `ReadWriteMany` (RWX).
-   `ReadWriteOncePod` (RWOP).

Support depends on the storage implementation.

### Ephemeral versus persistent storage

-   **Ephemeral storage:** generally tied to a Pod's lifecycle.
-   **Persistent storage:** designed to retain data independently of an
    individual Pod's lifecycle.

**Practical work:** Created a PVC and observed it become `Bound` using
the local StorageClass.

**Key takeaway:** PVs represent storage, PVCs request storage, and
StorageClasses define provisioning behavior.

------------------------------------------------------------------------

# Part III -- Workload Management and Security

## 9. Workload Controllers

Kubernetes provides different controllers for different workload
requirements.

  -----------------------------------------------------------------------
  Controller                          Purpose
  ----------------------------------- -----------------------------------
  Deployment                          Manages stateless applications

  StatefulSet                         Manages workloads requiring stable
                                      identity, ordering or persistent
                                      storage

  DaemonSet                           Runs Pods on selected nodes,
                                      commonly one per node

  Job                                 Runs a task to completion

  CronJob                             Creates Jobs on a schedule
  -----------------------------------------------------------------------

Examples of DaemonSet workloads include log collection agents and
node-level monitoring agents.

**Key takeaway:** Choose a workload controller based on the
application's behavior and requirements.

## 10. Health Checks and Resource Management

### Health probes

  -----------------------------------------------------------------------
  Probe                               Purpose
  ----------------------------------- -----------------------------------
  Liveness                            Checks whether an application is
                                      alive; repeated failure can trigger
                                      a container restart

  Readiness                           Checks whether an application is
                                      ready to receive traffic

  Startup                             Gives slower-starting applications
                                      time to initialize
  -----------------------------------------------------------------------

### Resource requests

Requests specify the amount of CPU or memory a container asks Kubernetes
to reserve for scheduling purposes.

### Resource limits

Limits define the maximum CPU or memory a container is permitted to use.

### ResourceQuota

A ResourceQuota limits aggregate resource consumption within a
namespace, such as:

-   Number of Pods.
-   Total CPU requests.
-   Total memory requests.
-   Total CPU limits.
-   Total memory limits.

**Key takeaway:** Probes help manage application health; requests and
limits manage container resources; ResourceQuotas control
namespace-level consumption.

## 11. Kubernetes Security

### RBAC

Role-Based Access Control determines what users and service accounts can
do within Kubernetes.

  -----------------------------------------------------------------------
  Resource                            Scope and purpose
  ----------------------------------- -----------------------------------
  Role                                Defines permissions within a
                                      namespace

  ClusterRole                         Defines cluster-scoped or reusable
                                      permissions

  RoleBinding                         Grants a Role or ClusterRole's
                                      permissions within a namespace

  ClusterRoleBinding                  Grants ClusterRole permissions at
                                      cluster level
  -----------------------------------------------------------------------

### ServiceAccounts

ServiceAccounts provide identities for applications and workloads
running inside Kubernetes.

### SecurityContext

SecurityContext controls security-related settings for Pods and
containers, including:

-   Running as a non-root user.
-   Disabling privilege escalation.
-   Dropping Linux capabilities.
-   Configuring seccomp profiles.

### Pod Security Admission

Pod Security Admission enforces workload security standards. Common
policy levels are:

-   Privileged.
-   Baseline.
-   Restricted.

**Key takeaway:** Kubernetes security involves controlling access,
limiting privileges and enforcing workload security standards.

## 12. Scheduling and Placement

The Kubernetes Scheduler selects suitable nodes for Pods. Scheduling
constraints help control placement.

  -----------------------------------------------------------------------
  Concept                             Purpose
  ----------------------------------- -----------------------------------
  `nodeSelector`                      Selects nodes using matching labels

  Node affinity                       Defines required or preferred node
                                      placement

  Taints                              Restrict which Pods can be
                                      scheduled on a node

  Tolerations                         Allow Pods to tolerate matching
                                      taints

  Pod affinity                        Places Pods near matching Pods

  Pod anti-affinity                   Places matching Pods apart

  Topology spread constraints         Distributes workloads across
                                      topology domains
  -----------------------------------------------------------------------

A toleration does not guarantee that a Pod will be scheduled on a
particular node.

**Key takeaway:** Scheduling controls where workloads run based on
resources, labels, constraints and infrastructure requirements.

------------------------------------------------------------------------

# Part IV -- Application and Cluster Operations

## 13. Helm

### What is Helm?

Helm is a package manager for Kubernetes. It simplifies the
installation, configuration, upgrade and management of Kubernetes
applications.

### Core concepts

-   **Chart:** a package containing files and templates used to define a
    Kubernetes application.
-   **Release:** an installed instance of a Helm Chart.
-   **Repository:** a location where Charts are distributed.
-   **`Chart.yaml`:** contains Chart metadata, including name and
    version.
-   **`values.yaml`:** contains default configuration values.
-   **Templates:** generate Kubernetes manifests using values and Helm's
    templating system.

Example values:

``` yaml
replicaCount: 2

image:
  repository: nginx
  tag: "1.27"
```

### Common operations

``` bash
helm install myapp ./myapp
helm upgrade myapp ./myapp
helm history myapp
helm rollback myapp 1
```

### Hooks

Helm hooks run actions at specific stages of a release lifecycle.
Examples include:

-   `pre-install`
-   `post-install`
-   `pre-upgrade`
-   `post-upgrade`
-   `pre-delete`
-   `post-delete`
-   `test`

### Other topics covered

-   Chart dependencies.
-   Chart packaging into `.tgz` archives.
-   Local Chart repositories.
-   Release revisions.
-   Rollbacks.
-   Helm troubleshooting and best practices.

**Practical work:** Created a `myapp` Chart, configured values,
installed and upgraded releases, rolled back revisions and packaged a
Chart.

**Key takeaway:** Helm makes Kubernetes deployments repeatable and
configurable.

## 14. Autoscaling

### What is autoscaling?

Autoscaling automatically adjusts application resources according to
demand or configured policies.

### Horizontal Pod Autoscaler (HPA)

HPA adjusts the number of Pod replicas based on observed metrics.

Important settings include:

-   Minimum replicas.
-   Maximum replicas.
-   Scaling metrics.
-   Target utilization.

### Metrics Server

Metrics Server collects current CPU and memory resource usage from
Kubernetes nodes and Pods.

``` bash
kubectl top pods
kubectl top nodes
```

HPA can use CPU utilization relative to CPU requests. Therefore, CPU
requests matter when configuring CPU-based HPA targets.

**Practical work:** Created an HPA, generated application load and
observed the replica count increase.

**Key takeaway:** HPA adjusts the number of application Pods based on
observed metrics and scaling policies.

## 15. Amazon EKS

### What is Amazon EKS?

Amazon Elastic Kubernetes Service is AWS's managed Kubernetes service.
AWS manages the Kubernetes control plane, while customers choose and
manage workload infrastructure according to their EKS configuration.

### Architecture and concepts covered

-   EKS control plane.
-   VPCs, subnets and Availability Zones.
-   Worker nodes.
-   Managed Node Groups.
-   EKS Auto Mode.
-   IAM and Kubernetes RBAC.
-   EKS Access Entries.
-   kubeconfig.
-   EKS add-ons.
-   Cluster provisioning options.

### Important add-ons

-   VPC CNI.
-   CoreDNS.
-   kube-proxy.
-   EBS CSI Driver.

### Connecting kubectl

``` bash
aws eks update-kubeconfig \
  --region us-east-1 \
  --name my-cluster
```

### Provisioning options

-   AWS Console.
-   AWS CLI.
-   eksctl.
-   Terraform.

**Learning status:** EKS concepts covered. Practical AWS cluster
creation was postponed to avoid unnecessary resource costs.

**Key takeaway:** EKS provides managed Kubernetes control-plane
infrastructure integrated with AWS services.

------------------------------------------------------------------------

# Part V -- Automation and Observability

## 16. GitOps

### What is GitOps?

GitOps is an operational approach where Git repositories contain the
desired configuration of an application or infrastructure environment.
Git acts as the source of truth.

### Core concepts

-   **Declarative configuration:** define the desired state.
-   **Reconciliation:** compare desired and actual state and work to
    synchronize them.
-   **Drift detection:** identify differences between Git configuration
    and the live environment.
-   **Self-healing:** correct certain live changes that conflict with
    the desired state, when enabled.

### Benefits

-   Version-controlled deployments.
-   Traceability.
-   Easier rollbacks.
-   Consistent environments.
-   Reduced manual deployment work.

**Key takeaway:** GitOps uses Git as the source of truth for managing
application and infrastructure configuration.

## 17. Argo CD

### What is Argo CD?

Argo CD is a declarative GitOps continuous delivery tool for Kubernetes.
It compares the desired state in Git with the actual state in the
cluster.

### Main components

  -----------------------------------------------------------------------
  Component                           Responsibility
  ----------------------------------- -----------------------------------
  API Server                          User-facing API, authentication and
                                      application operations

  Repository Server                   Retrieves Git configuration and
                                      renders manifests

  Application Controller              Compares desired and live state and
                                      coordinates synchronization
  -----------------------------------------------------------------------

### Application resource

An Argo CD Application defines the source and destination of an
application.

Important fields include:

-   `project`
-   `source.repoURL`
-   `source.targetRevision`
-   `source.path`
-   `destination.server`
-   `destination.namespace`
-   `syncPolicy`

### Synchronization

-   **Manual Sync:** a user initiates synchronization.
-   **Automated Sync:** Argo CD synchronizes automatically when
    configured conditions are met.

### Pruning and self-healing

-   **Pruning:** removes resources no longer defined in Git when enabled
    in the synchronization policy.
-   **Self-healing:** corrects certain live changes that differ from the
    desired Git state.

### Sync status versus health status

Sync status describes whether live state matches Git:

-   `Synced`
-   `OutOfSync`

Health status describes the condition of application resources:

-   `Healthy`
-   `Progressing`
-   `Degraded`
-   Other resource-specific states.

### AppProjects

AppProjects define boundaries around permitted repositories,
destinations and resource types.

### Sync Waves

Sync Waves control the order in which resources are applied.

Example:

-   Wave 0: ConfigMap.
-   Wave 1: Database.
-   Wave 2: Backend.

### Hooks

Important hooks include:

-   `PreSync`
-   `Sync`
-   `PostSync`
-   `SyncFail`

Hooks allow tasks to run at particular stages of synchronization.

### Notifications

Notifications inform teams about deployment events and application
status.

**Key takeaway:** Argo CD automates Kubernetes delivery using Git as the
source of truth.

## 18. Monitoring and Observability

### Monitoring

Monitoring is the process of collecting and examining information about
systems to understand their health and performance.

Examples:

-   CPU utilization.
-   Memory usage.
-   Network traffic.
-   Request count.
-   Error rate.
-   Response time.

### Observability

Observability is the ability to understand a system's internal behavior
through the information it exposes.

Three important observability signals are:

1.  Metrics.
2.  Logs.
3.  Traces.

Monitoring helps identify known conditions and problems. Observability
helps investigate system behavior, including unexpected problems.

**Key takeaway:** Monitoring helps show what is happening; observability
helps investigate why.

## 19. Prometheus and PromQL

### What is Prometheus?

Prometheus is an open-source monitoring and time-series database system.
It collects, stores and allows users to query metrics.

### Prometheus workflow

1.  An application or exporter exposes metrics.
2.  Prometheus periodically scrapes the metrics endpoint.
3.  Prometheus stores the collected time-series data.
4.  Users query the metrics using PromQL.
5.  Visualization tools such as Grafana can display the results.

### Metrics endpoints

Applications commonly expose metrics through an endpoint such as:

``` text
/metrics
```

### Exporters

Exporters expose metrics from systems that do not natively expose
Prometheus metrics.

  Exporter              Purpose
  --------------------- -----------------------------------
  Node Exporter         Linux system metrics
  Blackbox Exporter     Endpoint availability and probing
  Windows Exporter      Windows system metrics
  PostgreSQL Exporter   PostgreSQL database metrics

### Kubernetes metrics sources

  -----------------------------------------------------------------------
  Component                           Information provided
  ----------------------------------- -----------------------------------
  Kube State Metrics                  Kubernetes object state, such as
                                      Pod and Deployment status

  Metrics Server                      Current CPU and memory resource
                                      metrics

  cAdvisor                            Container resource usage
                                      information
  -----------------------------------------------------------------------

### PromQL

PromQL is Prometheus Query Language. It allows users to query and
analyze time-series metrics.

``` promql
up
```

Shows whether configured scrape targets are up.

``` promql
http_requests_total
```

Retrieves matching HTTP request counter series.

``` promql
http_requests_total{method="GET"}
```

Filters series using the `method` label.

### Functions covered

**`rate()`** calculates the average per-second increase of a counter
over a time window.

``` promql
rate(http_requests_total[5m])
```

**`increase()`** estimates the total counter increase over a time
window.

``` promql
increase(http_requests_total[5m])
```

**`sum()`** adds matching values.

``` promql
sum(rate(http_requests_total[5m]))
```

**`avg()`** calculates the average of matching values.

**Current learning status:** PromQL aggregation and further
Prometheus/Grafana topics remain in progress.

**Key takeaway:** Prometheus collects and stores metrics; PromQL lets us
query and analyze them.

## 20. Grafana

### What is Grafana?

Grafana is a visualization and dashboard platform. It connects to data
sources and displays information through dashboards and panels.

### Prometheus and Grafana

-   Prometheus collects and stores metrics.
-   Grafana queries Prometheus.
-   Grafana displays results through dashboards and visualizations.

### Dashboards

Dashboards organize related monitoring information in one place.
Examples include:

-   Kubernetes cluster overview.
-   Node CPU and memory.
-   Pod resource consumption.
-   Application request rates.
-   Error rates.

### Panels

Panels display individual visualizations, such as:

-   Time-series graphs.
-   Gauges.
-   Tables.
-   Stat panels.

**Key takeaway:** Grafana makes metrics easier to interpret by
presenting them visually.

------------------------------------------------------------------------

# Part VI -- Planned Topics

The following topics are part of the learning roadmap but have not yet
been fully covered.

## 21. Centralized Logging

Planned topics:

-   What logs are.
-   Application logs versus system logs.
-   Kubernetes container logs.
-   Centralized log collection.
-   Elasticsearch.
-   Fluent Bit or Fluentd.
-   Kibana.
-   EFK and ELK architectures.

## 22. Distributed Tracing

Planned topics:

-   Traces.
-   Spans.
-   Trace IDs.
-   Parent and child spans.
-   Request latency.
-   Identifying bottlenecks across microservices.

## 23. OpenTelemetry

Planned topics:

-   OpenTelemetry architecture.
-   Instrumentation.
-   Collectors.
-   Metrics.
-   Logs.
-   Traces.
-   Exporters and backends.

## 24. Kubernetes Administration

Planned topics:

-   Cluster upgrades.
-   Cluster backup and recovery.
-   etcd backup concepts.
-   Certificates.
-   Node maintenance.
-   Draining and cordoning nodes.
-   Cluster lifecycle management.

## 25. Advanced Kubernetes Concepts

Planned topics:

-   Custom Resource Definitions (CRDs).
-   Custom Controllers.
-   Kubernetes Operators.
-   Admission Controllers.
-   Mutating and Validating Admission Webhooks.

------------------------------------------------------------------------

# kubectl Quick Reference

  Command                                        Purpose
  ---------------------------------------------- ------------------------------------
  `kubectl get pods`                             List Pods
  `kubectl get deployments`                      List Deployments
  `kubectl get services`                         List Services
  `kubectl get nodes`                            List cluster nodes
  `kubectl get namespaces`                       List namespaces
  `kubectl describe pod NAME`                    Inspect a Pod
  `kubectl logs POD_NAME`                        View container logs
  `kubectl exec -it POD_NAME -- sh`              Open a shell in a container
  `kubectl apply -f file.yaml`                   Apply configuration
  `kubectl delete -f file.yaml`                  Delete resources defined in a file
  `kubectl scale deployment NAME --replicas=3`   Scale a Deployment
  `kubectl rollout status deployment/NAME`       Check rollout status
  `kubectl rollout history deployment/NAME`      View rollout history
  `kubectl rollout undo deployment/NAME`         Roll back a Deployment
  `kubectl top pods`                             View Pod resource usage
  `kubectl top nodes`                            View node resource usage

Use `-n NAMESPACE` when working in a specific namespace.

# Helm Quick Reference

  Command              Purpose
  -------------------- -------------------------------
  `helm repo add`      Add a Chart repository
  `helm repo update`   Update repository information
  `helm search repo`   Search repository Charts
  `helm create`        Create a Chart
  `helm lint`          Check a Chart
  `helm install`       Install a Release
  `helm upgrade`       Upgrade a Release
  `helm list`          List Releases
  `helm status`        Inspect a Release
  `helm history`       View Release history
  `helm rollback`      Roll back a Release
  `helm package`       Package a Chart
  `helm uninstall`     Remove a Release

# Important Concepts to Remember

1.  A Pod is the smallest deployable unit in Kubernetes.
2.  A Deployment manages ReplicaSets and stateless application Pods.
3.  A Service provides stable network access to Pods.
4.  An Ingress defines HTTP/HTTPS routing rules.
5.  A ConfigMap stores non-sensitive configuration.
6.  A Secret stores sensitive configuration and needs appropriate
    protection.
7.  A PVC requests persistent storage.
8.  Helm packages and manages Kubernetes applications.
9.  HPA adjusts the number of Pod replicas.
10. RBAC controls access to Kubernetes resources.
11. GitOps uses Git as the source of truth.
12. Argo CD synchronizes Git configuration with a Kubernetes cluster.
13. Prometheus collects and stores metrics.
14. Grafana visualizes information from Prometheus and other data
    sources.

# Learning Status

### Covered

-   Kubernetes fundamentals and architecture.
-   Workloads and Deployments.
-   Networking and Services.
-   Configuration and storage.
-   Security and scheduling.
-   Helm.
-   Autoscaling.
-   EKS concepts.
-   GitOps and Argo CD.
-   Monitoring foundations and initial Prometheus/PromQL concepts.

### In progress

-   Prometheus and Grafana.

### Planned

-   Centralized logging.
-   Distributed tracing.
-   OpenTelemetry.
-   Kubernetes administration.
-   Advanced Kubernetes concepts.
-   Integrated practical projects.

------------------------------------------------------------------------

## Notes and Future Updates

This reference is intended to grow as new topics are completed.

Future revisions can add:

-   More detailed explanations.
-   Practical labs and exercises.
-   YAML examples.
-   Architecture diagrams.
-   Troubleshooting notes.
-   Production considerations.
-   Links to official documentation.

**End of Version 1.0**
