# 2.Tekton Bundle

## Introduction 

As great as the Tekton Hub is, pulling and using task from there run the same security risk as pulling containers from public repos etc.   

!!! hnote "Jain writes"

      In the current scenario many of the Tekton users would be using the Tasks from catalog by first installing them in their cluster either by using kubectl or tkn command. Following this approach lowers the share-ability of the Pipelines as we need to share along the script or the Tasks which are used in the Pipeline. The current approach also introduces one more problem of immutability as the Task present on remote host can be changed at anytime and may change the behaviour and users installing the Task will get the newer Task which is incompatible with that Pipeline.



-------------------------------------------------------------------------------------------

## Supplementary Learning Materials

<iframe width="1120" height="630" src="https://www.youtube.com/embed/tQdI_9DebbI" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

### Additional links: 

* **Tekton Bundle** - <https://vinamra-jain.medium.com/publishing-tekton-resources-as-bundles-on-oci-registry-c4644d2af83f>
* **Tekton tasks in OCI** - <https://vinamra-jain.medium.com/referencing-tekton-tasks-from-oci-registry-5ff983d536e0>
* **Pipeline Tekton Bundles** - <https://tekton.dev/docs/pipelines/pipelines/#tekton-bundles>
* **Task Run Tekton Bundles** - <https://tekton.dev/docs/pipelines/taskruns/#tekton-bundles>
* **Kyverno Require Tekton Bundle** -<https://kyverno.io/policies/tekton/require-tekton-bundle/require-tekton-bundle/>

----------------------------------------------------------------------------------------------

## Scenario

1. Update ConfigMap feature-flags 
2. Push task to OCI Repo
3. Push Pipeline to OCI Repo
4. Push Pipelinerun to OCI Repo
5. List all resources in the bundle
6. list all the particular kind of resources in bundle 
7. See the YAML manifest of the particular resource
8. Install the Task on a Kubernetes cluster

??? solve "Solution"

      1.0 Update ConfigMap feature-flags
      OCI bundle support is still in alpha state so to enable it we need to edit the feature-flags ConfigMap by running the command. Change the value of enable-tekton-oci-bundles from false to true and save the ConfigMap and close the editor.

      ```
      kubectl edit configmap feature-flags -n tekton-pipelines
      ``` 

      2.0 Push img-scan-trivy task to OCI Repo

      ```
      $ tkn bundle push docker.io/mharrod/bundle:task -f img-scan-trivy.yaml
      ```

      3.0 Push scan-pipeline to OCI Repo
      ```
      $ tkn bundle push docker.io/mharrod/bundle:task -f scan-pipeline.yaml
      ```

      4.0 Push scan-pipeline-run to OCI Repo
      ```
      $ tkn bundle push docker.io/mharrod/bundle:task -f scan-pipeline-run.yaml
      ```

      alternatively you could have done this all at once


      ```
      $ tkn bundle push docker.io/mharrod/bundle:task -f img-scan-trivy.yaml -f scan-pipeline.yaml -f scan-pipeline-run.yaml 
      ```

      5.0 List all resources in the bundle

      ```
      $ tkn bundle list docker.io/mharrod/bundle:taskpipeline
      ```


      6.0 list all the particular kind of resources in bundle 

      ```
      $ tkn bundle list docker.io/mharrod/bundle:taskpipeline task task.tekton.dev/demo-task
      ```


      7.0 see the YAML manifest of the particular resource

      ```
      $ tkn bundle list docker.io/mharrod/bundle:taskpipeline task img-trivy-scan -o yaml

      ```

      8.0 install the Pipeline Kuberntest cluster
      Make sure you delete the previous task you created from the kubernetes cluster or are doing thos in a different namespace. 

      ```
      $ tkn bundle list docker.io/mharrod/bundle:taskpipeline task img-trivy-scan -o yaml | kubectl create -f 
      ```

----------------------------------------------------------------------------------

## Additional Challenges

* **Use a Bundle** - Rebuild the pipleine but using the task bundle approach.
* **Security Policy as code** - Add a Kyverno policy that requires the use of Tekton bundles when running in a Kubernetes environment. 