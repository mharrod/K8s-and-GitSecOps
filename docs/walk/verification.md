# 4.Image Verification in K8s(WIP)


## Introduction 

We will use Kyverno to validate that only signed images are deployed to Kubernetes.  Kyverno is an open-source policy engine for Kubernetes that allows you to define and enforce policies across your entire cluster. It allows you to define policies using Kubernetes native resources, such as ConfigMaps and Custom Resources, and it can be used to validate, mutate, and generate resources based on those policies. Kyverno provides a robust set of features for policy management, including:

* Policy validation: Kyverno allows you to define validation policies that can be used to check that resources conform to your desired state.
* Policy mutation: Kyverno allows you to define mutation policies that can be used to automatically modify resources as they are created or updated.
* Policy generation: Kyverno allows you to define generation policies that can automatically generate new resources based on existing resources.

Here's an example of how you could use Kyverno to enforce the use of signed images:

* Create a policy that defines the validation rules for image signatures. This policy could check for the presence of a signature and verify that the signature is valid and was created by a trusted source.
* Apply the policy to the relevant resources in the cluster. For example, you could apply the policy to all pods or deployments, or you could apply it to specific namespaces or labels.
* Implement the policy by creating a Kyverno admission webhook. This webhook will intercept all requests to create or update resources in the cluster and apply the policy to ensure that only signed images are allowed to run.
* Configure your image registry to provide the signature information for images that are pushed to it.

Once this policy is in place, any attempt to deploy an unsigned image will be rejected by the Kyverno admission webhook, ensuring that only signed images can run in the cluster.





## Supplementary Learning Material

<iframe width="1120" height="630" src="https://www.youtube.com/embed/M_-r6vUKevQ" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>



Work in progress ...


## Scenario

1. Sign Image with Cosign and verify signature
2. Sign Image and Verify signature
3. Generate, sign, and attach SBOM
4. Install Kyverno
5. Verify all images in registry
6. Create a deployment using an unsigned image
7. Create deployment using signed image 
8. Try (and fail) to deploy unsigned image to K8s


Work in progress ...

.
## Solution 

Work in progress ..



## Additional Challenges


