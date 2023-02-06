# 4.Production security testing

## Introduction 

Many recommend not doing security testing in production.  However, at the very least non, intrusive security testing should be done even in production.  Indeed, getting the exact parity of a production environment to staging is virtually impossible.  There will always be some issue or configuration that might get introduced during deployment to production (especially if the process still needs to be fully automated).  However, an even better approach is to use a blue/green deployment method in production and complete a deeper and more intrusive security testing regime in production than you normally could with a single production environment.

Blue/green deployment is a technique that can aid the security testing process by allowing teams to test changes and new releases in a staging environment before deploying them to production. This allows teams to test and validate changes in a non-production environment, which can help to identify and address any security issues before they are deployed to production.

Here's how blue/green deployment can aid the security testing process:

* **Isolation** - By deploying changes to a separate environment, teams can isolate the testing process from production and prevent any unintended consequences from occurring.

* **Controlled Rollback** - Blue/green deployment allows teams to quickly roll back to the previous version in case of any issues, this ensures that the system is not affected by any security issues, and the teams can fix it before rolling out to production.

* **Testing and Validation** - By deploying changes to a staging environment, teams can test and validate changes in a realistic environment, including security testing, which can help to identify and address any security issues before they are deployed to production.

* **Compliance** -  Blue/green deployment allows teams to test changes in a non-production environment, which can help to ensure that the changes are compliant with security standards and best practices before they are deployed to production.

By using blue/green deployment, teams can ensure that changes are thoroughly tested and validated in a non-production environment before they are deployed to production, which can help to improve the security of the system and reduce the risk of security breaches.

----------------------------------------------------------------------------------------------------

## Supplementary Learning Material

<iframe width="1120" height="630" src="https://www.youtube.com/embed/ArCu7Wi0AJY" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

#### Additional Links:

* **GitOps and Kubernetes book - Chapter 5** - <https://learning.oreilly.com/library/view/gitops-and-kubernetes/9781617297274/>
* **Argo Rollouts with Bluegreen** - <https://argoproj.github.io/argo-rollouts/features/bluegreen/>*
* **Tekton Task Bluegreen** - <https://hub.tekton.dev/tekton/task/blue-green-deploy>
* **Argo Rollouts with Bluegreen** - <https://www.infracloud.io/blogs/progressive-delivery-argo-rollouts-blue-green-deployment/>
* **Argo Rollouts with Bluegreen** - <https://blog.baeke.info/2021/11/01/kubernetes-blue-green-deployments-with-argo-rollouts/>

---------------------------------------------------------------------------------------

## Scenario

Work in progress ...


??? solve "Solution"
    Work in progress ...

-----------------------------------------------------------------------------------------

## Additional Challenges

Work in progress ...
