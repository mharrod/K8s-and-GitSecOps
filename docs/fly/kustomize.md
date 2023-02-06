# 1.Kustomize

## Introduction

Kustomize is a tool for customizing Kubernetes configurations, allowing you to manage variations of the same resource definitions. It provides a way to modify and extend existing resource manifests without changing the original files. Kustomize uses a Kubernetes-style declarative syntax and can be used to manage multiple environments such as development, testing, and production. It is commonly used as a component in a CI/CD pipeline for customizing resources for different environments, as well as for managing the separation of concerns between different teams.

Kustomize itself does not directly improve security, but it can help in implementing security best practices in a Kubernetes cluster by allowing you to manage configurations in a way that promotes security. Some ways that Kustomize can improve security include:

??? Harma "Separation of Concerns"
    Kustomize can help you separate different parts of a cluster into different configurations, making it easier to manage and secure different parts of your infrastructure.

??? Harma "Reducing Risk of Mistakes"
    With Kustomize, you can apply changes to resource definitions in a controlled and isolated manner, reducing the risk of making mistakes that could impact the security of your cluster.

??? Harma "Environment-Specific Configurations"
    Kustomize can help you manage environment-specific configurations, such as applying different security settings or configurations for development, testing, and production environments.

??? Harma "Better Visibility into Changes"
    By tracking changes to resource definitions through Kustomize, you can have better visibility into what changes have been made to your cluster and how they impact its security.

In summary, Kustomize can help you manage and secure your Kubernetes cluster by providing a way to apply and manage customizations in a controlled and transparent manner.

### Is Kustomize better than Helm?

The comparison between Kustomize and Helm is often a matter of personal preference and the specific use case. Both tools have their own strengths and weaknesses and the best choice depends on your requirements and goals.

Kustomize focuses on customizing existing resource definitions and is often used for overlaying changes on top of a base configuration. It is lightweight and does not require any additional resources or runtime components. Kustomize is well suited for teams that need to maintain a high degree of control over the resources in their cluster and want a simpler way to manage customizations.  On the other hand, Helm is a full-fledged package manager for Kubernetes and provides a way to manage the entire lifecycle of an application in a cluster, from deployment to upgrades and rollbacks. It is well suited for teams that need a more comprehensive solution for managing their applications and want a more streamlined experience.

Kustomize is a good choice if you need to manage customizations to existing resource definitions, while Helm is a good choice if you want a comprehensive solution for managing applications in a cluster.

In short, great idea to learn both!

-----------------------------------------------------------------------------------------

## Supplementary Learning Material

<iframe width="1120" height="630" src="https://www.youtube.com/embed/Twtbg6LFnAg" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

#### Additional Links:

* **Kustomize main site** <https://kustomize.io/> 
* **Gitops Cookbook - Chapter 4** - <https://learning.oreilly.com/library/view/gitops-cookbook/9781492097464/ch04.html#recipe_4_2>
* **Kustomize: getting Started** - <https://betterprogramming.pub/getting-started-with-kustomize-e9a84c4c2f97>
* **Leveraging Kustomize for Kubernetes manifests** - <https://learning.oreilly.com/library/view/leveraging-kustomize-for/9781098117078/>
* **Kustomize tooling** -  <https://www.densify.com/kubernetes-tools/kustomize>

-------------------------------------------------------------------------------------------

## Scenario

Work in progress . . .

??? solve "Solution"
    Work in progress . . .

-----------------------------------------------------------------------------------------

## Additional Challenges 

Work in progress . . .
