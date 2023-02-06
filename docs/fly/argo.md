# 2.ArgoCD

## Introduction

Declarative Continuous Delivery (CD) is a method of automating software releases that emphasize the use of declarative configuration files to define the system's desired state.  It is a way to specify how the software should be deployed rather than manually executing a series of commands to accomplish the deployment.  The declaration can include what software version should be deployed, the target environment, and any other relevant information.  These configuration files are then used to deploy and update the software consistently and predictably automatically.

One of the key advantages of a declarative CD is that it allows for version control of the configuration files, which enables teams to track changes and roll back to previous versions if necessary.  It also allows for easier collaboration among team members and makes it easier to automate the deployment process.

Declarative Continuous Delivery (CD) can help improve security in several ways:

??? Harma "Consistency and predictability"

     By using declarative configuration files to define the desired state of the system, declarative CD ensures that software is deployed in a consistent and predictable manner. This makes it easier to detect and address any security issues that may arise.

??? Harma "Version Control"

    Declarative CD enables teams to version control their configuration files, which allows them to track changes and roll back to previous versions if necessary. This makes it easy to identify and fix any security issues that may have been introduced with a new release.

??? Harma "Automation" 

    Declarative CD automates the deployment process, which reduces the risk of human error and makes it easier to detect and address security issues in a timely manner.

??? Harma "Isolation"
    Declarative CD often used in conjunction with containers and container orchestration tools such as Kubernetes, which provide a consistent and predictable environment for deploying software. This isolation helps to mitigate the risk of security breaches.

??? Harma "Compliance"
    Declarative CD can be integrated with security scanning tools to ensure that the deployed software is compliant with security standards and best practices. This helps to ensure that the software is secure and that any vulnerabilities are identified and addressed in a timely manner.

For our purposes, we will use ArgoCD and ArgoCD Image Updater for CD and image updates.  Argo CD is a GitOps-based continuous delivery tool that allows developers to manage and deploy applications in a Kubernetes cluster declaratively.  The Image Updater feature in Argo CD allows developers to quickly update container images in their deployed applications without manually updating the image version in the configuration files.  The Image Updater feature works by periodically checking for updates to container images in a specified container registry (e.g. Docker Hub, Quay) and comparing them to the images currently deployed in the cluster.  If an update is available, the Image Updater will automatically update the image in the cluster and update the configuration files in the Git repository to reflect the new image version.

-----------------------------------------------------------------------------------------------

## Supplementary Learning Material

<iframe width="1120" height="630" src="https://www.youtube.com/embed/fQ9846hRiFo" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

#### Additional Links:

* **GitSecops and kubernetes book - chapter 9** - <https://www.manning.com/books/gitops-and-kubernetes>
* Argocd in practice - https://learning.oreilly.com/library/view/argo-cd-in/9781803233321/
* **ArgoCD with Tekton** - <https://piotrminkowski.com/2021/08/05/kubernetes-ci-cd-with-tekton-and-argocd/>
* **Cloud native CI/CD pipeline** - https://medium.com/dzerolabs/using-tekton-and-argocd-to-set-up-a-kubernetes-native-build-release-pipeline-cf4f4d9972b0
* **Getting started with ArgoCD** - <https://argo-cd.readthedocs.io/en/stable/getting_started/>
* **ArgoCD, Tekton, and kaniko** - http://juggernaut.github.io/technology/kubernetes/ci/cd/gitops/argocd/tekton/2021/12/27/kubernetes-argocd-tekton.html
* **Tanzu and ArgoCD** - <https://tanzu.vmware.com/developer/guides/argocd-gs/>
* **Argo Image Updater** - https://medium.com/@jerome.decoster/argocd-image-updater-56cd94651393
* **Argo Image Updater** - <https://www.linkedin.com/pulse/automatic-image-update-using-argocd-satyam-kumar/?trk=articles_directory>

-------------------------------------------------------------------------------------------------

## Scenario

Work in progress ...


??? solve "Solution"
    Work in progress ...

------------------------------------------------------------------------------------------

## Additional Challenges

Work in progress ...





