# Fly(WIP)
 
## Introduction

This section looks at the role Kubernetes and everything else we have done up to this point can play a role in ensuring the security and efficacy of our release management process.  A release management process typically includes multiple environments: development, testing, staging, and production.   Security testing should be performed in each environment of the release management process to ensure that vulnerabilities are identified and addressed before the software is deployed to production.  The specific types of security testing that are performed in each environment may vary, but some approaches we will explore are depicted below: 


Kustomize 


<br>

```

                                                                                         Production
                                                                                       +---------------+
                                                                                       |   +--------+  |
                                                                                    +--|-->| Green  |  | -Automated
                     +---------------+     +---------------+     +---------------+  |  |   +--------+  | -SAST/DAST/IAST
                     |  Development  |---->|    Testing*   |---->|    Staging    |--|  |               | -Regression
                     +---------------+     +---------------+     +---------------+  |  |   +--------+  | -Runtime Controls
                        -SAST              -DAST/SAST               -SAST           +--|-->|  Blue  |  | -SRE Driven
                        -Automated         -Chaos Engineering       -Automated         |   +--------+  |
                                           -Test under Stress       -QA Driven         +---------------+
                                           -Expert Driven
                     
                                    *Suggest dedicated environment for security testing NOT shared with
                                       integration, performance, or non-security Chaos engineering

```
<br>


Many recommend not doing security testing in production.  However, at the very least non, intrusive security testing should be done even in production.  Indeed, getting the exact parity of a production environment to staging is virtually impossible.  There will always be some issue or configuration that might get introduced during deployment to production (especially if the process still needs to be fully automated).  However, an even better approach is to use a blue/green deployment method in production and complete a deeper and more intrusive security testing regime in production than you normally could with a single production environment.

??? Harma "How Blue/Green deployment can facilitate security testing in production"

    Blue/green deployment is a technique that can aid the security testing process by allowing teams to test changes and new releases in a staging environment before deploying them to production. This allows teams to test and validate changes in a non-production environment, which can help to identify and address any security issues before they are deployed to production.

    Here's how blue/green deployment can aid the security testing process:

    * **Isolation** - By deploying changes to a separate environment, teams can isolate the testing process from production and prevent any unintended consequences from occurring.

    * **Controlled Rollback** - Blue/green deployment allows teams to quickly roll back to the previous version in case of any issues, this ensures that the system is not affected by any security issues, and the teams can fix it before rolling out to production.

    * ***esting and Validation** - By deploying changes to a staging environment, teams can test and validate changes in a realistic environment, including security testing, which can help to identify and address any security issues before they are deployed to production.

    * **Compliance** -  Blue/green deployment allows teams to test changes in a non-production environment, which can help to ensure that the changes are compliant with security standards and best practices before they are deployed to production.

    By using blue/green deployment, teams can ensure that changes are thoroughly tested and validated in a non-production environment before they are deployed to production, which can help to improve the security of the system and reduce the risk of security breaches.

Here are some more of the scenarios we will explore

- TBD






