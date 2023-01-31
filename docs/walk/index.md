# Kubernetes 

## Introduction 

GiSectOps in Kubernetes refers to a method of managing infrastructure and applications by using Git as the single source of truth for configuration and deployment.  It is a way to automate and simplify the management of Kubernetes clusters by using Git as the central place to store, version, and track all the configuration files for the application and infrastructure.  In this scenario, we will work towards building the following environment.
<br>
<br>

```
                                                        +-----------------------------------------------------------------------+
                                                        |                       KUBERNETES CLUSTER                              |
                            +-----------{#}-+           |   +--------------------+                                              |
                            |   GIT REPO    |           |   | ArgoCD Namespace   |                    +------------------{#}-+  |
                            | +-----------+ | Monitors  |   |  +--------------+  |                    |     APP Namespace    |  |
                            | | App Code  | |<----------|---|--|   ArgoCD     |  |    Update/Apply    |  +----------------+  |  |
                            | +-----------+ |           |   |  +--------------+  +-------+     +----->|  |  StudentBook   |  |  |
                            |               |           |   |                    |       |     |      |  +----------------+  |  |
                            | +-----------+ |           |   |  +--------------+  |       |     |      |                      |  |
                            | | Manifests | |<----------|---|--| ArgoCD Image |  |   +-----{#}-----+  +----------------------+  |
                            | +-----------+ |   Rewrite |   |  |   Updater    |  |   |   Kyverno   |                            |
                            |               |     Tag   |   |  +--------------+  |   | Enforcement |                            |
                            +---------------+           |   |      |             |   +-------------+                            |
                                                        |   +--------------------+                                              |
                                                        |          |                                                            |
                                                        +-----------------------------------------------------------------------+
                                +---------{#}-+                    |
                                |     CI      |                    | Monitors           {#} = Securtiy Inspection
                                | +--------+  |                    |
                                | | Tekton |  |                    v
                                | +--------+  |            +-------------------------------------{#}---+
                                |             |  Publish   |                REGISTRY                   |
                                | +--------+  |   Image    | +-----------+ +-----------+ +-----------+ |
                                | | Kaniko |--|----------->| |  Harbor   | |   Local   | | DockerHub | |
                                | +--------+  |            | +-----------+ +-----------+ +-----------+ |
                                |             |            |                                           |
                                +-------------+            +-------------------------------------------+
```
<br>

With GitSecOps for Kubernetes, developers can set up an automated pipeline that periodically checks for updates to container images in a specified container registry (e.g., Docker Hub, Quay) and automatically updates the images in the cluster using tools such as Kaniko. This ensures that applications are always running on the latest version of the container images, which can include security patches and other important updates.  Further, using tools like Kyverno, one can define and enforce policies across the entire cluster, including policies for ensuring the use of signed images.

Here is a summary of some of the ways that GitSecOps can be used to improve the security of applications deployed on Kubernetes:

??? Harma "Secure the application code"

    Use Git and code review processes to ensure that the application code is secure and follows best practices. This can include using static code analysis tools to catch security vulnerabilities, as well as implementing access controls to manage access to the codebase.

??? Harma "Use Git for configuration management"
    Use Git to manage the configuration of your Kubernetes cluster and applications. This can help ensure that changes to the configuration are tracked and can be easily rolled back if necessary.

??? Harma "Use Git for continuous integration and delivery"

    Use Git and CI/CD tools to automate the build, test, and deployment process for your applications. This can help ensure that applications are deployed quickly and safely, while also making it easier to roll back deployments if necessary.

??? Harma "Monitor and track security vulnerabilities"

    Use Git and security tools to track and monitor security vulnerabilities in your applications and infrastructure. This can help you identify and fix vulnerabilities before they are exploited.

## Scenarios Explored

* **K8s, Kind and Security Scanning** - Set-up a local Kubernetes development clusters with Kind and deploy images using Kaniko.
* **Build Images with Kaniko** - Using Kaniko to build images without Docker Daemon
* **Image Verification in K8s** - Detecting and enforcing the use of signed images in K8s.
* **Networking and Environments** - Leveraging Kubernetes namespaces and networking for environment seperation.

