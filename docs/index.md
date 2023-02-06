# Start 

## Introduction

This workshop curates the experience of setting up a cloud-native but vendor-agnostic GitSecOps pipeline that can be used to build, deliver, and manage microservices at any scale. Of course, this will not be a production-grade implementation but more of a reference architecture. It will allow the reader to explore the various facets of GitSecOps pipeline development and give them insight into how they may wish to pursue such an endeavor in their environments.

At a high level, this is what we are looking to achieve:


```
                                           Skopeo
                                        Kaniko/Buildah
                                        Podman/Docker                      Nexus           Helm         Kind
                             GitHub        Tekton          Tekton        DockerHub        ArgoCD      Kubernetes
                                |             |               |              |              |             |
                                |             |               |              |              |             |
                           ------------------------------------------------------------------------------------>
                                                DevOps - Creating Value and Availability
                          +-----------+  +-----------+  +-----------+  +-----------+  +-----------+  +-----------+
                          |   CODE    |  |  BUILD    |  |    TEST   |  | ARTIFACTS |  |  DEPLOY   |  |  OPERATE  |
                          +-----------+  +-----------+  +-----------+  +-----------+  +-----------+  +-----------+
                                               SecOps - Creating Trust & Confidence
                           <-------------------------------------------------------------------------------------
                                 |             |               |             |              |               |
                                 |             |               |             |              |               |
                              Semgrep        Grype            ZAP          Snyk        Trivy/Snyk          Falco
                             SonarQube      ggshield        SemGrep                                     GLF Stack
                             TruffleHog     Halolint       SonarQube                                    Metasploit
                               Snyk       Tekton Chains    Trivy/Snyk                                     Trivy
                                           DockerBench                                                     ZAP
                                            KubeBench
                                           Trivy/Snyk
                                           Nmap/Nessus
                                            Nikto/Zap
                                              Syft

```

??? Harma "Additional Reading: Theory of GitSecOps"

    GitSecOps is a practice that combines the version control capabilities of Git with security tools and processes to improve the security of an organization's software and infrastructure. By integrating security into the software development workflow, GitSecOps helps organizations ensure that their products are secure from the start, rather than trying to retroactively fix security issues after the fact.  Some of the value of a GitSecOps pipeline are:

    - **Provides a clear record of all changes made to the codebase** - Git keeps track of every change made to the codebase, along with the username of the person who made the change. This makes it easy to track down the source of any security issues that may arise, and to roll back changes if necessary.

    - **Leverages code reviews** - Code review tools allow teams to review each other's code and catch any security issues before they are merged into the main codebase.

    - **Incorporates automated testing tools** -  Automated testing tools can run a suite of tests on the codebase to ensure that it meets security standards, and can alert developers if any issues are found.

    - **Makes it easier to manage and track access to sensitive code and infrastructure** -  By using Git and other tools to enforce access controls, organizations can ensure that only authorized users have access to sensitive information, reducing the risk of data breaches or other security incidents.

    Overall, GitSecOps helps organizations improve the security of their products by integrating security into the software development process, providing a clear record of changes, enforcing security policies through code review and automated testing, and managing access to sensitive information. By adopting GitSecOps practices, organization


    **Implementing GitSecOps**
    There is no one "best" way to implement GitSecOps, as the specific practices and tools that will work best for your organization will depend on your unique needs and goals. However, there are some general best practices that you can follow to help ensure the success of your GitSecOps efforts:

    - **Involve security experts in the process** -  Make sure that your GitSecOps team includes individuals with expertise in security. They can help identify and prioritize potential security risks and advise on the best tools and practices to address them.

    - **Automate as much as possible** -  Automating security checks and processes can help ensure that they are consistently and accurately applied, freeing up time for developers to focus on other tasks.

    - **Regularly review and update your GitSecOps practices** - As your organization's needs and the security landscape evolve, it is important to regularly review and update your GitSecOps practices to ensure that they are still effective.

    - **Foster a culture of security** - Encourage all team members to prioritize security in their work and make it a part of the company's overall culture. This can involve providing security training and making it easy for team members to report any potential security issues they encounter.


**Caveats and Exceptions** 

1. This is a work in progress and is in a constant state of change and improvement. Please check back regularly.
2. This is not meant to guide a production-ready environment. This workshop is experimental. As a result, many of the outcomes are clunky and theoretical.    
3. This is not meant to be a panacea. There are dozens of different tooling configurations that could be used. The tools used were picked based on several factors including but not limited to: 1) Acceptance by the CNCF, 2) Availability of books, tutorials, etc to expand learning, 3) Examples of use in production, 4)no/low cost)
4. Not much time (if any) is spent teaching readers how to use the tools. Rather, we curate a selection of learning resources that participants should read through before completing any particular section.
5. This workshop focuses on building a local environment in KIND only, but could be adapted to other settings by the participant.

## Principles of design 

When building out this reference pipeline, we use the following design principles:

* **Security by design** - Security is at the forefront of everything we do.  It is not something we come back and bolt on later.
* Kubernetes first - Where possible, leverage tooling/mechanisms that use Kubernetes CRDs (reduces need to learn multiple configuration languages etc.)
* **Open source** - Whenever possible, we use and contribute to open source projects.
* **Automate** - After learning to do something by hand, we aim to automate.
* **Everything as code** - We aim to make it so everything we do is done as code.
* **Cloud Native** - We are building this to operate on ANY Cloud, whether public or private.
* **Vendor Agnostic** - The pipeline, with minor, tweaks should work everywhere.

## Progressions

#### Crawl, Walk, Run, Fly 

Initially, there are 4 phases to work through: Crawl, Walk, Run, Fly.  The "crawl, walk, run" methodology is an approach to introducing new concepts or capabilities gradually and incrementally.  It involves starting with a basic, foundational level (crawl), then slowly building up to more advanced levels (walk), and ultimately achieving a level of proficiency or independence (run and fly ).  It is essentially a way of introducing a new feature by starting with a basic prototype (crawl), then gradually adding more functionality and refining the design (walk), and ultimately releasing a fully-featured version of the feature (run and fly).
<br>
<br>

!!! Hnote inline end "Note" 

    Each stage above has 3-4 scenarios  that are structured as follows:

    * **Introduction** - High-level overview of the discovery.
    * **supplementary Reading** - Links to tutorials and other information that provide knowledge on how to complete the scenario.
    * **Scenario** - The problem or task at hand.  Readers use knowledge from the supplementary reading to complete the task. 
    * **Solution** - A potential solution to the problem presented in the scenario.
    * **Challenge** - An optional challenge that allows participants to test their understand of the concepts discovered.

**Crawl:**  

  In this phase we explore the basics of implementing a simple application first as a "local" (not running in containers) application and then later running in a container.  Throughout these scenarios, we explore how to securely select and validate images before deploying.  These are the scenarios explored during this stage:

* **Local Application** - How does one validate the code of an application and deploy locally?
* **Containerized Application** - What is the best way to choose and securley use containerized Applications and Images?
* **Image From Scratch** - How should one securely build a container image from scratch?
* **Image Integrity** - How does one validate the integrity of an image?
<br>


**Walk:** 

We will explore using several deployment methods to deploy and manage containers in a Kubernetes environment.  Basic security mechanisms for kubernetes will also be explored. 

* **Kind, Kaniko, and K8s** - Set-up a local Kubernetes development clusters with Kind and deploy images using Kaniko.
* **Build Images with Kaniko** - Using Kaniko to build images without Docker Daemon
* **Image Verification in K8s** - Detecting and enforcing the use of signed images in K8s.
* **Networking and Environments** - Leveraging Kubernetes namespaces and networking for environment seperation.

<br>


**Run:** 

Reader will leverage Tekton to build a repeatable cloud native CI/CD system to deploy the application on Kubernetes.

* **Pipeline as code** - Use pipeline as code to automate a security scan of a container image. 
* **Tekton Bundles** - Create and published signed Tekton bundles.
* **Tekton Builds** - Using Tekton and Kaniko to build, sign an push an image to an OCI Registry. 
* **Tekton Triggers** - Using Tekton triggers to automatically update application on Git commit. Also using Kyverno to enforce the use of verified images and Tekton bundles on K8s.
<br>

**Fly:**

For this final step, we are going to go deeper into the end-to-end securing of the GitSecOps pipeline. 

* **Using Kustomize** - How to use Kustomize to simplify K*s deployment in a declarative way 
* **ArgoCD** - Incorporating ArgoCD to provide declarative continuous deployment.
* **Dedicated security testing** - Implementing and using a dedicated security namespace in the testing phase. 
* **Security testing in production** - Using blue/green testing deployment approaches to facilitate rigorous security testing in production. 


### 2.2 Capstone
The Capstone project is an opportunity to apply all the principles discovered during the workshop to a greenfield application.  Readers are encouraged to use the application provided, but there is also the opportunity to use their own application. 

### 2.3 Explorations 
Explorations allow readers to dive deeper into GitSecops, Microservices security, and much more.  This is done by leveraging the book.info reference architecture and creating an exploration that others can follow. Once you are done with the previous stages, you will endeavor to write your own discovery and add it to the growing list of Explorations.

  








