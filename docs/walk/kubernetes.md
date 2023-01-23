# 1.K8s, Kind and Security Scanning

## Introduction

Kubernetes (often shortened to "K8s") is an open-source container orchestration system.  It was originally developed by Google and is now maintained by the Cloud Native Computing Foundation.  Kubernetes allows developers to easily deploy, scale, and manage containerized applications in a clustered environment.  Kubernetes provides a set of abstractions that are used to define, deploy, and manage containerized applications.  These abstractions include pods, the smallest deployable units in Kubernetes, and are used to group one or more containers.  Services, which provide stable network connections to pods and Replication Controllers, ensure that the desired number of pod replicas are running at all times.

Kubernetes also provides load balancing, automatic scaling, and self-healing to ensure that applications are highly available and can handle changing traffic patterns.  Additionally, Kubernetes can be integrated with other tools and services, such as monitoring and logging, to provide a complete solution for managing containerized applications.

To develop with Kubernetes clusters locally, we will use KIND Kubernetes IN Docker (kind).  Overall, KIND is a powerful tool for local Kubernetes development because it is easy to set up and use, provides an isolated environment for testing and debugging, and integrates well with other tools.  Other options, such as minikube, work equally as well. 

## Supplementary Learning Materials

<iframe width="1120" height="630" src="https://www.youtube.com/embed/s_o8dwzRlu4" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

Additional Links:

* Kubernetes and Kind - <https://www.baeldung.com/ops/kubernetes-kind>
- Kubernetes and Kind - <https://phoenixnap.com/kb/kubernetes-kind>
- Kubernetes cluster development with kind - <https://faun.pub/creating-a-kubernetes-cluster-for-development-with-kind-189df2cb0792>
* Kind with podman - <https://www.linkedin.com/pulse/kind-podman-ubuntu-2004lts-ludger-pottmeier>
* Kind & K8s lab - <https://cloudyuga.guru/hands_on_lab/kind-k8s>
* Juiceshop on Kubernetes - <https://networkandcode.hashnode.dev/run-owasp-juice-shop-as-a-kubernetes-service>
* Ingress for Kubernetes - <https://kind.sigs.k8s.io/docs/user/ingress/#ingress-nginx>
* Kubescape - <https://www.howtogeek.com/devops/how-to-scan-for-kubernetes-vulnerabilities-with-kubescape/>
* Security scanning for K8s - <https://mattermost.com/blog/the-top-7-open-source-tools-for-securing-your-kubernetes-cluster/#kubeaudit>
* Helm - <https://helm.sh/docs/intro/quickstart/>

## Scenario 

1. Create a named Cluster called JuiceShop
2. Create or copy deployment manifest and service manifest and sabe for Juiceshop (provided above)
3. Scan Juice Manifests with Kubescape
4. Install Juice shop with manifests
5. Complete a security scan of the Kubernetes environment (Include: Kubesecurity bench, Kubeaudit, Kubescape)
6. Fetch Juice-Shop Helm Chart and scan with Checkov and Helm lint 
7. Install JuiceShop Application with Helm chart
8. Install StudentBook Using manifests

??? Harma "Juice-Shop Manifests"

        You are free to write these on your own or you can just copy and paste

        ```
        cat <<'EOF'>> juice-shop-manifests.yaml
        kind: Deployment
        apiVersion: apps/v1
        metadata:
          name: juice-shop-manifests
        spec:
          template:
            metadata:
              labels:
                app: juice-shop-manifests
            spec:
              containers:
              - name: juice-shop-manifests
                image: bkimminich/juice-shop
          selector:
            matchLabels:
              app: juice-shop-manifests
        ---
        kind: Service
        apiVersion: v1
        metadata:
          name: juice-shop-manifests
        spec:
          type: NodePort
          selector:
            app: juice-shop-manifests
          ports:
          - name: http
            port: 8000
            targetPort: 3000
        ---
        EOF

        ```

??? solve "Solution"
      1.0 Create a Cluster named Juice with NGNIX Support

      1.1 Install Kind

      ```
      curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.17.0/kind-linux-amd64
      chmod +x ./kind
      sudo mv ./kind /usr/local/bin/kind

      # If you are going to use podman then you need to add it as an external provider in your ~/.bashrc or ~/.zshrc file
      $ export KIND_EXPERIMENTAL_PROVIDER=podman

      # If you like, you can alos add an alias to the docker command to your ~/.bashrc or ~/.zshrc file
      alias docker='podman'

      # Other things you may need to do can be found here https://kind.sigs.k8s.io/docs/user/rootless/
      ```

      1.2 Create Cluster default Kind

      ```
      cat <<EOF | kind create cluster --name studentbook --config=-
      kind: Cluster
      apiVersion: kind.x-k8s.io/v1alpha4
      nodes:
      - role: control-plane
        image: kindest/node:v1.25.3@sha256:f52781bc0d7a19fb6c405c2af83abfeb311f130707a0e219175677e366cc45d1
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
      <br>
      -----------------------------------------------------------------------------------------------
      2.0 Create JuiceSHop Manifests (do not deploy yet)

      ```
      cat <<'EOF'>> juice-shop-manifests.yaml
      kind: Deployment
      apiVersion: apps/v1
      metadata:
      name: juice-shop-manifests
      spec:
      template:
        metadata:
          labels:
            app: juice-shop-manifests
        spec:
          containers:
          - name: juice-shop-manifests
            image: bkimminich/juice-shop
      selector:
        matchLabels:
          app: juice-shop-manifests
      ---
      kind: Service
      apiVersion: v1
      metadata:
      name: juice-shop-manifests
      spec:
      type: NodePort
      selector:
        app: juice-shop-manifests
      ports:
      - name: http
        port: 8000
        targetPort: 3000
      ---
      EOF
      ```
      <br>
      -----------------------------------------------------------------------------------------------
      3.0 Scan Juice Manifests with Kubescape

      ```
      curl -s https://raw.githubusercontent.com/kubescape/kubescape/master/install.sh | /bin/bash
      kubescape scan framework nsa k8s/*.yaml
      ```
      <br>
      -----------------------------------------------------------------------------------------------
      4.0 Deploy Juiceshop via manifests 

      4.1 Create deployment

      ```
       kubectl create -f juice-shop-manifests.yaml
      ```

      4.2 Watch service 
      ```
      kubectl get po --watch
      ```

      4.3  If want to browse by local pod IP then do following

      ```
      # Check service endpoints
        kubectl get svc juice-shop
      # get internal IP 
      kubectl get no -o wide | awk '{print $6}'
      ```
      4.4 If you want to browse via local host then do the following 
      ```
      kubectl port-forward svc/juice-shop-manifests 8080:8000  
      ```
      <br>
      -----------------------------------------------------------------------------------------------
      5.0 Complete a security scan of the Kubernetes environment 


      5.1 Using Kubescape 

      ```
      curl -s https://raw.githubusercontent.com/kubescape/kubescape/master/install.sh | /bin/bash

      # scan everything
      $ kubescape scan --submit --enable-host-scan --format-version v2 --verbose 

      # scan Manifests 
      $ kubescape scan framework nsa *.yaml

      # scan specific file 
      $ kubescape scan framework nsa --exclude-namespaces kube-system,kube-public

      ```

      5.2 Kubesecurity Bench 

      ```
      kubectl apply -f - <<EOF
      ---
      apiVersion: batch/v1
      kind: Job
      metadata:
        name: kube-bench
      spec:
        template:
          metadata:
            labels:
              app: kube-bench
          spec:
            hostPID: true
            containers:
              - name: kube-bench
                image: docker.io/aquasec/kube-bench:v0.6.10
                command: ["kube-bench"]
                volumeMounts:
                  - name: var-lib-etcd
                    mountPath: /var/lib/etcd
                    readOnly: true
                  - name: var-lib-kubelet
                    mountPath: /var/lib/kubelet
                    readOnly: true
                  - name: var-lib-kube-scheduler
                    mountPath: /var/lib/kube-scheduler
                    readOnly: true
                  - name: var-lib-kube-controller-manager
                    mountPath: /var/lib/kube-controller-manager
                    readOnly: true
                  - name: etc-systemd
                    mountPath: /etc/systemd
                    readOnly: true
                  - name: lib-systemd
                    mountPath: /lib/systemd/
                    readOnly: true
                  - name: srv-kubernetes
                    mountPath: /srv/kubernetes/
                    readOnly: true
                  - name: etc-kubernetes
                    mountPath: /etc/kubernetes
                    readOnly: true
                    # /usr/local/mount-from-host/bin is mounted to access kubectl / kubelet, for auto-detecting the Kubernetes version.
                    # You can omit this mount if you specify --version as part of the command.
                  - name: usr-bin
                    mountPath: /usr/local/mount-from-host/bin
                    readOnly: true
                  - name: etc-cni-netd
                    mountPath: /etc/cni/net.d/
                    readOnly: true
                  - name: opt-cni-bin
                    mountPath: /opt/cni/bin/
                    readOnly: true
            restartPolicy: Never
            volumes:
              - name: var-lib-etcd
                hostPath:
                  path: "/var/lib/etcd"
              - name: var-lib-kubelet
                hostPath:
                  path: "/var/lib/kubelet"
              - name: var-lib-kube-scheduler
                hostPath:
                  path: "/var/lib/kube-scheduler"
              - name: var-lib-kube-controller-manager
                hostPath:
                  path: "/var/lib/kube-controller-manager"
              - name: etc-systemd
                hostPath:
                  path: "/etc/systemd"
              - name: lib-systemd
                hostPath:
                  path: "/lib/systemd"
              - name: srv-kubernetes
                hostPath:
                  path: "/srv/kubernetes"
              - name: etc-kubernetes
                hostPath:
                  path: "/etc/kubernetes"
              - name: usr-bin
                hostPath:
                  path: "/usr/bin"
              - name: etc-cni-netd
                hostPath:
                  path: "/etc/cni/net.d/"
              - name: opt-cni-bin
                hostPath:
                  path: "/opt/cni/bin/"
      EOF

      ```

      5.2.1 Wait for a few seconds for the job to complete

      ```
      $ kubectl get pods
      ```

      5.2.2 The results are held in the pod's logs
      ```
      kubectl logs kube-bench . . .
      ```

      5.3 Kube Hunter 

      5.3.1 Installing kube-hunter to check from inside the cluster

      ```
      $ kubectl create -f https://raw.githubusercontent.com/aquasecurity/kube-hunter/master/job.yaml
      $ kubectl get pods
      $ kubectl logs <kube-hunter-pod-name>

      ```
      <br>
      -----------------------------------------------------------------------------------------------
      6.0 Fetch Juice-Shop Helm Chart and scan with Checkov and Helm lint 

      6.1 Install Helm

      ```
       mkdir helm && cd helm
       curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash
       helm repo update

      ```

       6.2 Add the helm repo for Juice-shop

      ```
      $ helm repo add juice https://charts.securecodebox.io
      ```

      6.3 Inspect Juiceshop Helm Chart 
      ```
      $ helm fetch juice/juice-shop 
      $ tar -xvf juice-shop-3.15.2.tgz
      $ cd 
      $ helm lint juice-shop/  
      $ pip install checkov  
      $ checkov -d juice-shop/ --framework helm
      ```

      7.0 Install JuiceShop Application with Helm chart

      ```
      $ helm install juice-shop-helm juice/juice-shop
      ```


## Additional Challenges
1. **Additional Vulnerability Scanning** - Install Scanbox and run the security scans we did in previous section with it
2. **Automate Testing** - Write a bash script to automate the security testing 
3. **Take the helm** - Write and audit a helm chart fot the StudentBook application



















