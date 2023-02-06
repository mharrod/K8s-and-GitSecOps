# 4.Triggering Tekton

# Introduction 

It is now time to bring the automation processes one step further and automatically start those pipelines as soon as some changes are pushed to your code repository.  To do this, we will use Tekton Triggers. Triggers is a project of Tekton Pipelines that provides a set of building blocks for creating custom event-driven pipelines, with support for listening to events from a variety of sources, such as Git repositories, message queues, and webhooks. further. With Triggers, we can now s to start our pipeline automatically based on changes.

This is advantageous to security because it adds automation to security tasks. By automating security tasks and enforcing security policies, Tekton Triggers can help you reduce the risk of security breaches, improve the security of your applications and infrastructure, and ensure that your organization stays compliant with security regulations and standards.

In addition to this, we are going to use Kyverno to enforce the use of signed Tekton builds, signed images etc before they can be deployed to the Kubernetes cluster.

--------------------------------------------------------------------------------

## Supplementary learning material  

<iframe width="1120" height="630" src="https://www.youtube.com/embed/zVkumUImIao" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

#### Additional links

* **Tekton Triggers** - <https://tekton.dev/docs/triggers/>
* **Tekon Triggers Gettind started** - <https://tekton.dev/docs/getting-started/triggers/>
* **Tekton Triggers Tutorial** - <https://github.com/tektoncd/triggers/blob/main/docs/getting-started/README.md>
* **NGROK** - <https://ngrok.com/download>
* **Secure CI/CD with Kyverno** - <https://www.cncf.io/blog/2022/09/14/protect-the-pipe-secure-ci-cd-pipelines-with-a-policy-based-approach-using-tekton-and-kyverno/>
* **Verify Images** - <https://boxboat.com/2021/12/06/secure-supply-chains-kyverno/>
* **Kyverno Policies** - <https://kyverno.io/policies/?policytypes=Tekton>

--------------------------------------------------------------------------------------------


## Scenario

### Task 1 - Create Tekton Trigger for StudentBook

1. Install Tekton Triggers
2. Install and configure Ngrok (if using local cluster)
3. Create pipeline to be triggered
4. Create the trigger
5. Configure incoming webhooks
6. Expose route and make publicly available
7. Configure Github repository 
8. Trigger the pipeline 


??? solve "Solution Task 1"
    Work in progress ...

#### Task 2 - Use Kyverno to enforce policy on Tekton

Work in progress . . .

??? solve "Solution Task 2"
    Work in progress ...

----------------------------------------------------------------------------------------------

## Additional Challenges

Work in progress 


