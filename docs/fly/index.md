# Fly(WIP)
 
## Introduction

it is time to put all that knowledge into practice and create a pipeline to deploy an application directly from a GitHub repository into a given cluster. 

This section looks at the role Kubernetes and everything else we have done up to this point can play a role in ensuring the security and efficacy of our release management process.  A release management process typically includes multiple environments: development, testing, staging, and production.   Security testing should be performed in each environment of the release management process to ensure that vulnerabilities are identified and addressed before the software is deployed to production.  The specific types of security testing that are performed in each environment may vary, but some approaches we will explore are depicted below: 

```

                                Testing
                         +---------------------+
                         |  +---------------+  |                              Production
                      +--|--|  Performance  |--|--+                         +---------------+
                      |  |  +---------------+  |  |                         |   +--------+  |
                      |  |                     |  |                      +--|-->| Green  |  |
+---------------+     |  |  +---------------+  |  |   +---------------+  |  |   +--------+  |
|  Development  |---->|--|--|      QA       |--|--|-->|    Staging    |--|  |               |
+---------------+     |  |  +---------------+  |  |   +---------------+  |  |   +--------+  |
   -SAST              |  |                     |  |      -QA Driven      +--|-->|  Blue  |  |
   -Automated         |  |  +---------------+  |  |      -SAST              |   +--------+  |
                      +--|--|    Security   |--|--+                         +---------------+
                         |  +---------------+  |                              -Automated
                         +---------------------+                              -SAST/DAST/IAST
                             -DAST/SAST                                       -Regression
                             -Chaos Engineering                               -Runtime Controls
                             -Test under Stress                               -SRE Driven
                             -Expert Driven

                                    *Suggest dedicated environment for security testing NOT shared with
                                       integration, performance, or non-security Chaos engineering

```
<br>
In addition to performing security testing in every environment, we also suggest bifurcating or splitting the pipeline in the Test environments to create a dedicated security testing environment.  We practiced this in the wlk:network-environments scenario) Having a separate CI/CD pipeline for security testing can bring several benefits

## Scenarios

Here are the scenarios we will explore::

* **Using Kustomize** - How to use Kustomize to simplify K*s deployment in a declarative way 
* **ArgoCD** - Incorporating ArgoCD to provide declarative continuous deployment.
* **Dedicated security testing** - Implementing and using a dedicated security namespace in the testing phase. 
* **Security testing in production** - Using blue/green testing deployment approaches to facilitate rigorous security testing in production. .





