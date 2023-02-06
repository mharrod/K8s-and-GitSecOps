# 3.Build a container

Tekton and Kaniko are a powerful combination for building and pushing containers in a Kubernetes environment. Tekton provides a flexible and scalable CI/CD platform that automates and manages the container build and push process, while Kaniko is a secure and efficient tool for building containers designed specifically for running in a Kubernetes environment and eliminates the need for a Docker daemon and can run in a read-only environment. By using Tekton and Kaniko together, you can increase efficiency, improve security, and gain more flexibility in the container build and push process. 

By adding Tekton Chains and Cosign to the mix we can provide an added layer of security to the software development process by verifying the authenticity and integrity of the image being deployed. By adding signing to the container we can ensure that only signed and verified images are deployed and run in our kubernetes environment. 

------------------------------------------------------------------------------------

## Supplementary Learning Material

<iframe width="1120" height="630" src="https://www.youtube.com/embed/EHZA_kMHmYE" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

#### Additional Links:

* **Kaniko and Tekton** - <https://tekton.dev/docs/how-to-guides/kaniko-build-push>
* **Kaniko and Tekton** - <https://ibm.github.io/tekton-tutorial-openshift/lab1/2_build-and-push-image-using-kaniko/>
* **CI Pipelines with Tekton** - <https://www.arthurkoziel.com/creating-ci-pipelines-with-tekton-part-2/>*
* **Tanzu Building a K8s container** - <https://tanzu.vmware.com/developer/guides/tekton-gs-p2/>
* **Tekton Chains provenance** - <https://tekton.dev/docs/chains/signed-provenance-tutorial/>
* **Tekton Chains** - <https://cd.foundation/blog/2022/10/18/tekton-chains/>
* **Google SLSA** - <https://security.googleblog.com/2021/06/verifiable-supply-chain-metadata-for.html>
* **Golang signed provenance** - https://pxp928.com/posts/2022/03/tekton-golang-pipeline-with-signed-provenance-slsa-level-2/

------------------------------------------------------------------------------------------

## Scenario

Run the pipeline
1. Generate a key pair
2. Set-up authentication
3. Patch tekton chains 
4. Install the git-clone and Kaniko tasks
5. Create service account apply the secret with your Docker credentials
6. Apply the pipeline
7. Create the pipeline run
8. Use the pipeline run name to monitor
9. Verify signature with cosign
10. Find provenance in Rekor

??? solve "Solution"
    Work in progress . . .

--------------------------------------------------------------------------------

## Additional Challenges

Work in progress . . .
