# 1.Pipeline as Code

Pipelines as code is a software development practice where CI/CD pipelines are defined and managed in code form, rather than being manually created in a continuous integration tool. This means that the pipeline configuration is stored in source control, just like any other software artifact, and can be versioned, tracked, and audited.

With pipelines as code, CI/CD pipelines can be defined in a declarative, self-contained manner.  This declarative approach makes it easy to understand, maintain, and scale the pipeline. The pipeline definition is stored in a file format such as YAML or JSON, which can be automatically processed and executed by a continuous integration tool, such as Tekton Pipelines.

In the scenarios for the run section we will use Tekton Pipelines.  Tekton is an open-source framework for creating CI/CD pipelines that are flexible, scalable, and reusable. It is designed to be cloud-native and can run on any platform, including on-premise, public clouds, and hybrid clouds. With Tekton Pipelines, you can automate the process of building, testing, and deploying your applications, ensuring that the entire pipeline is consistent, reproducible, and efficient. 

"Pipeline as code" helps improve security in several ways:

??? Harma "Version control"
    By storing pipelines in code, they can be versioned, tracked, and audited. This makes it easier to identify and revert any changes that may have introduced security vulnerabilities.

??? Harma "Automated reviews"
    When pipelines are code, they can be automatically reviewed by security tools and scanned for vulnerabilities, ensuring that pipelines are secure before they are deployed.

??? Harma "Improved Collaboration"
    Improved collaboration: When pipelines are code, they can be shared, reviewed, and modified by multiple teams, making it easier to maintain secure pipelines.

??? Harma "Consistency" 
    With pipelines as code, the pipeline definition is standardized, making it easier to maintain consistent security practices across the entire pipeline.

??? Harma "Compliance"
    By automating pipelines, organizations can ensure that security policies and regulations are followed consistently, making it easier to comply with security standards and regulations.

In conclusion, "pipeline as code" provides a more secure, automated, and auditable approach to CI/CD pipelines, making it easier to maintain secure, compliant, and reliable pipelines. For this first scenario, we will build a simple pipeline and use it to automate some of the security tests that we did in earlier sections. Please note, Tekton has a steep learning curve, we strongly strongly suggest that you work through the following book before ven trying anything going forward 

--------------------------------------------------------------------------------

## Supplementary Learning  

<iframe width="1120" height="630" src="https://www.youtube.com/embed/7mvrpxz_BfE" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

### Additional links: 

* **Building Ci/CD Systems Using Tekton book** - <https://www.packtpub.com/product/building-cicd-systems-using-tekton/9781801078214> 
* **VSCode plugin for Tekton** - <https://github.com/redhat-developer/vscode-tekton>
* **Tekton tutorial** - <https://tekton.dev/docs/dashboard/tutorial/>
* **Tekton hello world** - <https://tanzu.vmware.com/developer/guides/tekton-gs-p1/>
* **IBM Tekton Tutorial** - <https://developer.ibm.com/tutorials/build-and-deploy-a-docker-image-on-kubernetes-using-tekton-pipelines/>
* **Tekton and Trivy** - <https://github.com/MoOyeg/trivy-tekton-example>
* **Tekton and Trivy** - <https://github.com/mbharatk/trivy-tekton-pipeline>
* **Cloud Native CI/CD with Tekton** - <https://martinheinz.dev/blog/47>

-------------------------------------------------------------

## Scenario

1. Install Tekton, Tekton Cli, and Tekton Dashboard 
2. Create a Task that does the following:
   a. Uses Skopeo to download the Juiceshop image
   b. Download Trivy and scan JuiceShop image
3. Create a Pipeline
4. Create a Pipeline Run 
5. Apply the Task, Pipeline, and Pipeline Run
6. Run the Pipeline
7. Delete Pipeline


#### Note Tested using the following versions
- Client version: 0.28.0
- Pipeline version: v0.42.0


??? solve "Solution"

      1.1 Install Tekton, 

      ```
      kubectl apply --filename https://storage.googleapis.com/tekton-releases/pipeline/latest/release.yaml

       kubectl apply --filename https://storage.googleapis.com/tekton-releases/pipeline/previous/v0.42.0/release.yaml

      ```
      1.2 Install Tekton Dashboard
      ```
      kubectl apply --filename https://storage.googleapis.com/tekton-releases/dashboard/latest/release-full.yaml
      ```

      1.3 Monitor the Installation
      ```
      kubectl get pods --namespace tekton-pipelines --watch
      ```

      1.4 Install Tekton cli

      ```
      # Get the tar.xz
      curl -LO https://github.com/tektoncd/cli/releases/download/v0.29.0/tkn_0.29.0_Linux_x86_64.tar.gz
      # Extract tkn to your PATH (e.g. /usr/local/bin)
      sudo tar xvzf tkn_0.29.0_Linux_x86_64.tar.gz -C /usr/local/bin/ tkn

      ```
      1.5 Check set-up

      ```
      $ tkn version

      ```
      1.6 Access the Tekton Dashboard 

      ```
      kubectl port-forward -n tekton-pipelines service/tekton-dashboard 9097:9097
      ```
      Browse to http://localhost:9097

      2.0 Create a task

      In this case, we are going to create files locally and run, rather than the EOF Apply method we have been using.  This will make completing the second task easier.

      2.1 Create folders for scenario in location of your choice
      ```
      $ mkdir bundle
      $ cd bundle
      ```

      2.2 Create task
      ```
      cat > img-scan-trivy.yaml <<EOF
      apiVersion: tekton.dev/v1beta1
      kind: Task
      metadata:
        name: img-scan-trivy
      spec:
        params:
          - name: image-url
            description: "The location of image to scan on Container Registry <server>/<namespace>/<repository>:<tag>"
            default: "true"
          - name: SKOPEO_IMAGE
            default: quay.io/containers/skopeo:v1.1.0
          - name: IMAGE_FROM_TLS_VERIFY
            default: "true"
          - name: TRIVY_IMAGE
            default: docker.io/aquasec/trivy
        volumes:
          - name: oci-image
            emptyDir: {}
        steps:
          - name: pull-image
            image: $(params.SKOPEO_IMAGE)
            volumeMounts:
              - mountPath: /var/oci
                name: oci-image
            script: |
              IMAGE_FROM=$(params.image-url)
              REGISTRY_SERVER_FROM=$(echo "${IMAGE_FROM}" | awk -F / '{print $1}')
              IMAGE_TO="oci:/var/oci/image"
              IMAGE_FROM_TLS_VERIFY=$(params.IMAGE_FROM_TLS_VERIFY)
              echo "Tagging ${IMAGE_FROM} as ${IMAGE_TO}"
              echo "skopeo copy --src-creds=xxxx --src-tls-verify=${IMAGE_FROM_TLS_VERIFY} docker://${IMAGE_FROM} ${IMAGE_TO}"
              set +x
              skopeo copy ${IMAGE_FROM_CREDS} --src-tls-verify=${IMAGE_FROM_TLS_VERIFY} docker://${IMAGE_FROM} ${IMAGE_TO}
          - name: scan-image
            image: $(params.TRIVY_IMAGE)
            volumeMounts:
              - mountPath: /var/oci
                name: oci-image
            script: |
                set -e
                PATH_TO_IMAGE="/var/oci/image"
                echo -e "Trivy Security Scan image in registry"
                trivy image --exit-code 1 --input ${PATH_TO_IMAGE}
                my_exit_code=$?
                echo "Scan exit code :--- $my_exit_code"
                if [ ${my_exit_code} == 1 ]; then
                    echo "Trivy scanning completed.  Vulnerabilities found."
                    exit 1
                else
                  echo "Trivy scanning completed.  vulnerabilities not found."
                fi
      EOF
      ```

      3.0 Create and apply a pipeline 

      ```
      cat > scan-pipeline.yaml <<EOF
      apiVersion: tekton.dev/v1beta1
      kind: Pipeline
      metadata:
        name: scan-pipeline
      spec:
        params:
          - name: image-url
            description: "The location of image to scan on Container Registry <server>/<namespace>/<repository>:<tag>"
            default: ""
          - name: SKOPEO_IMAGE
            default: quay.io/containers/skopeo:v1.1.0
          - name: IMAGE_FROM_TLS_VERIFY
            default: "true"
          - name: TRIVY_IMAGE
            default: docker.io/aquasec/trivy
        tasks:
          - name: image-scan
            taskRef:
              name: img-scan-trivy
            params:
              - name: image-url
                value: $(params.image-url)
              - name: SKOPEO_IMAGE
                value: $(params.SKOPEO_IMAGE)
              - name: IMAGE_FROM_TLS_VERIFY
                value: $(params.IMAGE_FROM_TLS_VERIFY)
              - name: TRIVY_IMAGE
                value: $(params.TRIVY_IMAGE)
      EOF
      ```

      4.0 Create and apply a pipeline run 

      ```
      cat > scan-pipelinerun.yaml <<EOF
      apiVersion: tekton.dev/v1beta1
      kind: PipelineRun
      metadata:
        name: scan-pipelinerun
      spec:
        pipelineRef:
          name: scan-pipeline
        
        params:
        - name: image-url
          value:  ""
        - name: SKOPEO_IMAGE
          value: quay.io/containers/skopeo:v1.1.0
        - name: IMAGE_FROM_TLS_VERIFY
          value: "true"
        - name: TRIVY_IMAGE
          value: docker.io/aquasec/trivy
      EOF
      ```

      5.0 Run the pipeline

      5.1 Install the task
      ```
      kubectl apply -f img-scan-task.yaml
      ```

      5.2 Apply the pipeline
      ```
      kubectl apply -f scan-pipeline.yaml
      ```


      5.3 Apply the Pipeline run
      ```
      kubectl apply -f scan-pipelinerun.yaml
      ```


      6.0 Run the pipeline

      ```
      $ tkn pipeline start scan-pipeline
      ```

      6.1 Monitor the pipeline 
      ```
      $ tkn pipelinerun logs scan-pipeline-run
      ```

      6.2 Enumerate all pipeline artifacts

      ```
      $ tkn task list -n <namespace>
      $ tkn pipeline list -n <namespace>
      $ tkn pipelinerun list -n <namespace>
      ```

      7.0 Delete Pipeline

      ```
      $ tkn pipelinerun delete scan-pipeline-run -n <namespace>
      $ tkn pipeline delete scan-pipeline -n <namespace>
      $ tkn task delete img-trivy-scan -n <namespace>

      ```

------------------------------------------------------------------

## Additional Challenges

* **Using Tekton Hub** - Update the pipeline to use the Tekton Hub to incorporate the Syft reusable task to generate an SBOM. 

