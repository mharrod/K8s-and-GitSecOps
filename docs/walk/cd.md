# 3.Declarative Continuous Delivery 


## Introduction

Declarative Continuous Delivery (CD) is a method of automating software releases that emphasize the use of declarative configuration files to define the system's desired state.  It is a way to specify how the software should be deployed rather than manually executing a series of commands to accomplish the deployment.  The declaration can include what software version should be deployed, the target environment, and any other relevant information.  These configuration files are then used to deploy and update the software consistently and predictably automatically.

One of the key advantages of a declarative CD is that it allows for version control of the configuration files, which enables teams to track changes and roll back to previous versions if necessary.  It also allows for easier collaboration among team members and makes it easier to automate the deployment process.

Declarative Continuous Delivery (CD) can help improve security in several ways:

??? Harma "Consistency and predictability"

     By using declarative configuration files to define the desired state of the system, declarative CD ensures that software is deployed in a consistent and predictable manner. This makes it easier to detect and address any security issues that may arise.

??? Harma "Version Control"

    Declarative CD enables teams to version control their configuration files, which allows them to track changes and roll back to previous versions if necessary. This makes it easy to identify and fix any security issues that may have been introduced with a new release.

??? Harma "Automation" 

    Declarative CD automates the deployment process, which reduces the risk of human error and makes it easier to detect and address security issues in a timely manner.

??? Harma "Isolation"
    Declarative CD often used in conjunction with containers and container orchestration tools such as Kubernetes, which provide a consistent and predictable environment for deploying software. This isolation helps to mitigate the risk of security breaches.

??? Harma "Compliance"
    Declarative CD can be integrated with security scanning tools to ensure that the deployed software is compliant with security standards and best practices. This helps to ensure that the software is secure and that any vulnerabilities are identified and addressed in a timely manner.

For our purposes, we will use ArgoCD and ArgoCD Image Updater for CD and image updates.  Argo CD is a GitOps-based continuous delivery tool that allows developers to manage and deploy applications in a Kubernetes cluster declaratively.  The Image Updater feature in Argo CD allows developers to quickly update container images in their deployed applications without manually updating the image version in the configuration files.  The Image Updater feature works by periodically checking for updates to container images in a specified container registry (e.g. Docker Hub, Quay) and comparing them to the images currently deployed in the cluster.  If an update is available, the Image Updater will automatically update the image in the cluster and update the configuration files in the Git repository to reflect the new image version.


## Supplementary Learning Material

<iframe width="1120" height="630" src="https://www.youtube.com/embed/fQ9846hRiFo" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

#### Additional Links:

* ArgoCD - <https://argo-cd.readthedocs.io/en/stable/>
* Harbor - <https://goharbor.io/>
* kind and Ingress - <https://kind.sigs.k8s.io/docs/user/ingress/>
* Local Harbor Install <https://serverascode.com/2020/04/28/local-harbor-install.html>
* ArgoCD Install - <https://tanzu.vmware.com/developer/guides/argocd-gs/>
* Triy & Harbor - <https://artifacthub.io/packages/helm/trivy-operator/harbor-scanner-trivy/0.28.0>
* Kubernetes Namespaces <https://www.aquasec.com/cloud-native-academy/kubernetes-101/kubernetes-namespace/>
* Kubens and Kubectx  for nmespace/cluster management - <https://github.com/ahmetb/kubectx>
* Harbor and Helm - <https://itnext.io/need-a-container-image-registry-and-helm-chart-repository-go-harbor-b0c0d4eafd3b>
* Harbor and Helm <https://mannimal.blog/2019/07/31/using-harbor-and-kubeapps-to-serve-custom-helm-charts/>



## Scenario 

1. Create Kind cluster with NGNIX  Support 
2. Install Harbor (make sure to enable support for Trivy)
3. Create new project called juice (easiest to do via the GUI)
4. Use Docker Client to push juiceshop to Harbor
5. Try Pulling Juice Image from Harbor
6. Upload Juice Helm Chart to Harbor
7. Install ArgoCD and Argo Cli 
8. Create an Argo Application
9. Deploy StudentBook via ArgoCD 


??? Harma "Solution"

      1.0 Create Kind cluster with NGNIX  Support  


      1.1 Install Kind with Ingress Support

      ```
      $ cat <<EOF | kind create cluster --name juice --config=-
      kind: Cluster
      apiVersion: kind.x-k8s.io/v1alpha4
      nodes:
      - role: control-plane
        kubeadmConfigPatches:
        - |
          kind: InitConfiguration
          nodeRegistration:
            kubeletExtraArgs:
              node-labels: "ingress-ready=true"
        extraPortMappings:
        - containerPort: 80
          hostPort: 80
          protocol: TCP
        - containerPort: 443
          hostPort: 443
          protocol: TCP
      EOF
      ```

      1.2 Install Ingress 

      1.21. If using Ngnix

      ```
      $ kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml
      ```

      1.2.2 If using Contour

      ```
      $ kubectl apply -f https://projectcontour.io/quickstart/contour.yaml
      $ kubectl patch daemonsets -n projectcontour envoy -p '{"spec":{"template":{"spec":{"nodeSelector":{"ingress-ready":"true"},"tolerations":[{"key":"node-role.kubernetes.io/master","operator":"Equal","effect":"NoSchedule"}]}}}}'
      ```

      2.0 Install Harbor (make sure to enable support for Trivy)


      2.1 Add Harbor Helm repo and install 
      ```
      $ helm repo add harbor https://helm.goharbor.io

      # Install in separate namespace
      $ helm install local-harbor harbor/harbor \
        --create-namespace \
        --namespace harbor \
        --set clair.enabled=false \
        --set trivy.enabled=true

      # Or install in same namespace
        $ helm install local-harbor harbor/harbor \
        --set clair.enabled=false \
        --set trivy.enabled=true
      ```

      2.2 Accessing the site 

      ```
      # Final configd
      - Add pointer 127.0.0.1 core.harbor.domain /etc/hosts
      - Browse https://core.harbor.domain Must access via https or login will fail
      - login: admin password: Harbor12345
      ```

      3.0 Create a new project via the gui


      ```
      1. Add a robot account
      2. Set Policy to scan on push and prevent deployment of vulnerable images 
      3. Create project called "juice"
      ```

      4.0 Use Docker Client to push Juiceshop to Harbor

      ```
      $ docker login username (will prompt for password )
      $ docker pull bkimminich/juice-shop
      $ docker tag bkimminich/juice-shop core.harbor.domain/juice/juice:v1
      $ docker push core.harbor.domain/[projectname]/juice:v1
      ```

      5.0 Try Pulling Juice Image from Harbor

      ```
      $ docker pull core.harbor.domain/redsmith/juice:v1      

      Error response from daemon: unknown: current image with 52 vulnerabilities cannot be pulled due to configured policy in 'Prevent images with vulnerability severity of "Low" or higher from running.' To continue with pull, please contact your project administrator to exempt matched vulnerabilities through configuring the CVE allowlist.

      ```

      6.0 Upload Juice Helm Chart to Harbor 


      6.1 Fetch Helm Chart
      ```
      $ helm fetch juice/juice-shop 
      ```

      6.2 Upload via GUI

      ```
      -Upload via GUI
      ```

      7.0 Install ArgoCD and Argo Cli

      7.1 Install ArgoCD

      ```
      $ kubectl create namespace argocd
      $ kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
      $ watch kubectl get pods -n argocd
      $ kubectl port-forward svc/argocd-server -n argocd 8080:443

      ```

      7.2 Install ArgoCD Cli
      ```
      # Download the binary
      $ curl -sLO https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64


      # Make binary executable
      $ chmod +x argocd-linux-amd64

      # Move binary to path
      $ sudo mv ./argocd-linux-amd64 /usr/local/bin/argo

      # Test installation
      $ argo version --short
      ```
      7.3 Login and Change password

      ```
      $ argocd login localhost:8080
      Note: Again, as with the UI, you will need to accept the server certificate error.

      ## To get the password
      $ kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d

      $ argocd account update-password

      kubectl port-forward svc/argocd-server -n argocd 8080:443
      Site now available at https://localhost:8080


      ```

      7.3 Set Target K8s cluster

      ```
      $ argocd cluster add juice 
      ```
      8.0 Create an Argo Application
      ```
      work in progress . . .
      ```

      9.0  Deploy Juiceshop via ArgoCD 
      ```
      Work in progress . . .
      ```
      

## Additional Challenges

* **Use Clair instead of Trivy** - Change the default scanner to Clair or another scanning engine
* **Add certs to Harbor** - Set-up Hsrbor so that it uses proper certs and TLS 
* **Use Kaniko** - Build and push the Juiceshop Image to Harbor with Kaniko 







