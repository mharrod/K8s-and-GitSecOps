# Run

## Introduction

In the run stage, we will explore using Pipelines as Code(PaC) to build and deploy our application to our Kubernetes cluster.  PaC refers to defining the steps of a continuous integration and delivery (CI/CD) pipeline in code form, rather than using a graphical user interface (GUI) to configure the Pipeline.  This allows the pipeline configuration to be stored in version control along with the rest of the application code, making it easier to track changes to the Pipeline over time and roll back to previous versions if necessary.  We will also strongly emphasize automating all the security testing we have done up to this point with the CI portion of the Pipeline. 
<br>
<br>

```
            +-----------------------------------------------------------------------------+ +------------------+ +---------------------+
            |                            Establish Trust                                  | |   Verify Trust   | |   Maintain Trust    |
            +-----------------------------------------------------------------------------+ +------------------+ +---------------------+

                    +--------------------------------------------------------+   +-----------------------------------------------+
                    |        Static Automated Security Testing               |   |           DAST & Runtime Security             |
                    +-----------------------------------------------------+>-+   +----------------------------------+------------+
                    |                  |                    |            |                     |                   |
                    |                  |                    |            |                     |                   |
            +-------+------------------+--------------------+------------|---------------------+-------------------|-------+
            |       |    Vulnerability |   Testing          |            |                     |                   |       |
            +-------+---+--------------+--+-----------------+--+---------|-------+-------------+---+---------------|---+---+
                    |   |              |  |                 |  |         |       |             |   |               |   |
                    v   v              v  v                 v  v         v       v             v   v               v   v
            +----------------------------------------------------------------------------+ +-------------------+ +-------------------+
            |   Tekton                                                                   | |  ArgoCD           | | Observability     |
            | +---------------+  +---------------+  +---------------+  +---------------+ | | +---------------+ | | +---------------+ |
            | |     Code      |  |     Build     |  |     Test      |  |   Artifacts   | | | |    Deploy     | | | |    Operate    | |
            | +---------------+  +---------------+  +---------------+  +---------------+ | | +---------------+ | | +---------------+ |
            |                                                                            | |                   | |                   |
            +----------------------------------------------------------------------------+ +-------------------+ +-------------------+
                                                ^                                                  ^
                                                |                                                  |
                                        +-------+-------+                                  +-------+-------+
                                        | Code App Repo |                                  |  Config Repo  |
                                        +---------------+                                  +---------------+
```
<br>

We will use Tekton as the CI for Pipelines as Code for our purposes.  Tekton is an open-source project for creating cloud-native continuous integration and delivery (CI/CD) pipelines.  It is designed to be easy to use and flexible, focusing on building and deploying containerized applications.  However, for the most part, everything we do could be ported to the CI tool of your choice.  

Some of the benefits of using Tekton for building CI/CD pipelines include:

??? Harma "Cloud-native"

    Tekton is designed to be run in a cloud environment, making it easy to build and deploy containerized applications to the cloud.

??? Harma "Customizable"

    Tekton provides a set of modular building blocks that can be combined and customized to create a CI/CD pipeline that meets the specific needs of your organization.

??? Harma "Kubernetes native"
    Tekton is built on top of Kubernetes, making it easy to use with other Kubernetes-based tools and platforms.

??? Harma "Flexible"
    Tekton can be used with a variety of different tools and technologies, including container registry platforms, source control systems, and build and deployment tools.

??? Harma "Scalable"
    Tekton is designed to scale to meet the needs of large organizations, with the ability to run pipelines concurrently and handle large numbers of builds and deployments.

??? Harma "Security Automation"
    Tekton pipelines can be configured to include security scanning as a step in the pipeline process. This can be done by including tasks that run security scanning tools and services, such as Aqua Security, Snyk, and Anchore, as part of the pipeline.

Here are the scenarios we will explore::

* **Pipeline as code** - Use pipeline as code to automate a security scan of a container image. 
* **Tekton Bundles** - Create and published signed Tekton bundles.
* **Tekton Builds** - Using Tekton and Kaniko to build, sign an push an image to an OCI Registry. 
* **Tekton Triggers** - Using Tekton triggers to automatically update application on Git commit. Also using Kyverno to enforce the use of verified images and Tekton bundles on K8s.
