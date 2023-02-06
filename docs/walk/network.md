# 4.Networking and Environments

The software deployment process involves different environments with different purposes, ranging from local development to production release. The CI/CD process involves promoting code changes through the Dev, QA, Stage, and Prod environments, each of which has its own testing and configuration requirements. The goal is to ensure that code changes are thoroughly tested in preproduction environments before being released to production, where they can impact actual customers.

Kubernetes can be used to manage the deployment of applications across different environments by creating separate namespaces for each environment. For example, you can create a namespace for "development", another for "stage", and another for "production". Then you can create deployment objects, service objects, and other Kubernetes resources within each namespace to represent the application components.

In this scenario, we are going to create two clusters.  One cluster will be called "preprod" with the namespaces QA and sectest.  The second cluster will be named "prod"  and include the name stages for stage and prod.  We will then experiment with how we can create separation for these environments and leverage Kyverno to enforce network policies.

------------------------------------------------------------------------------------

## Supplementary Learning material

<iframe width="1120" height="630" src="https://www.youtube.com/embed/RP-jSB2eclk" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

Additional Links:

* **Gitops and Kubernetes Chapter 3** - <https://www.manning.com/books/gitops-and-kubernetes>
* **Calico and Network Policy** - <https://projectcalico.docs.tigera.io/security/kubernetes-network-policy>
* **Redhat K8s Network Policy** - <https://cloud.redhat.com/blog/guide-to-kubernetes-egress-network-policies>
* **Calico k8s Policy basics** - <https://docs.projectcalico.org/security/tutorials/kubernetes-policy-basic>
* **K8s Network Policy** - <https://kubernetes.io/docs/concepts/services-networking/network-policies/>

------------------------------------------------------------------------------

## Scenario

Kind (as with minikube and most other local Kubernetes environments) uses kindnetd as the default for the CNI.  This CNI is too basic to complete the network policy exercise in the chapter.  As a result, we will install Calico as our CNI.  If you notice in the configuration files above, we set disableDefaultCNI: true.  This setting ensured that the default CNI was not installed and that we could layer on our own. See the following link if you want to better understand CNI https://www.tigera.io/learn/guides/kubernetes-networking/kubernetes-cni


#### Task 1 - QA and Sectest
1. Deploy a cluster named prepod witj Calico CNI installed 
2. Switch to prepod cluster and create environment Namespaces qa and sectest.
3. Deploy a Curl application  to both QA and sectest 
4. Deploy an NGNix webserver in QA
5. try to curl from both qa and sectest
6. Apply a netwrok policy to prevent all pods outside qa from accessing qa
7. Try to curl NGNIX webserver from each environment (sectest fails and qa passes)

??? Harma "kubetx and kubens"

        Kubetx and Kubens make interacting with multiple clusters and namespaces easier. There are ways to do this with kubectl, but these tools/plugins make it easier.  The two tools are:

        1. **Kubectx** - kubectx is a tool to switch between contexts (clusters) on kubectl faster
        2. **kubens** - is a tool to switch between Kubernetes namespaces (and configure them for kubectl) easily

        Here is the link to the git repo https://github.com/ahmetb/kubectx

        There are many ways to install these tools(depending on if you want auto-complete etc.).  For this walkthrough, we keep  it simple and do the following;

        ```
        $ wget https://raw.githubusercontent.com/ahmetb/kubectx/master/kubectx
        $ wget https://raw.githubusercontent.com/ahmetb/kubectx/master/kubens
        $ chmod +x kubectx kubens
        $ sudo mv kubens kubectx /usr/local/bin
        ```


??? solve "Solution Task 1" 

     1.O Deploy a cluster named prepod and a cluster named prod with Calico CNI installed

     1.1 Create First Cluster 

     Here is the configuration file for the first cluster 

     ```
     cat <<EOF | kind create cluster --name preprod --config=-
     kind: Cluster
     apiVersion: kind.x-k8s.io/v1alpha4
     nodes:
       - role: control-plane
       - role: worker
     networking:
       podSubnet: 10.240.0.0/16
       serviceSubnet: 10.110.0.0/16
       disableDefaultCNI: true
     EOF

     ```

     1.2 Create Second Cluster 

     Here is the configuration for the second cluster 
     ```
     cat <<EOF | kind create cluster --name prod --config=-
     kind: Cluster
     apiVersion: kind.x-k8s.io/v1alpha4
     nodes:
       - role: control-plane
       - role: worker
     networking:
       podSubnet: 10.241.0.0/16
       serviceSubnet: 10.111.0.0/16
       disableDefaultCNI: true
     EOF

     ```

     1.3 Review and make sure the clusters are set up correctly

     ```
     $ kind get clusters

     ```

     1.4  Make sure you are on the preprod cluster 

     ```
     kubectx kind-preprod
     ```

     1.5 install the Tigera operator on the cluster.

     ```
     $ kubectl create -f https://docs.projectcalico.org/manifests/tigera-operator.yaml
     ```


     1.6 Apply the configuration

     ```
     kubectl apply -f - <<EOF
     apiVersion: operator.tigera.io/v1
     kind: Installation
     metadata:
       name: default
     spec:
       calicoNetwork:
         ipPools:
           - blockSize: 26
             cidr: 10.240.0.0/16
             encapsulation: VXLANCrossSubnet
             natOutgoing: EnabledInstall Kyverno and apply dedault deny-all policy to all namespaces created 
     ```

     1.7 Verify installation 
     We may verify the installation of Calico by listing running pods in the calico-system namespace. Below is a sample result.  Please note, it can take several minutes (i have seen as long as ten) for kubernetes to actual apply and get Calico up and running.

     ```
     kubectl get pod -n calico-system
     NAME                                       READY   STATUS    RESTARTS   AGE
     calico-kube-controllers-777bf44dbc-6x4lc   1/1     Running   0          25m
     calico-node-5pmzw                          1/1     Running   0          25m
     calico-node-jnmrx                          1/1     Running   0          25m
     calico-typha-59747bfc5-zgmlk               1/1     Running   0          25m
     csi-node-driver-74wg4                      2/2     Running   0          12m
     csi-node-driver-gszqh                      2/2     Running   0          15m

     ```


     2.0 Switch to prepod cluster and create environment Namespaces qa and sectest.


     2.1 Make sure we are on the preprod cluster 
     ```
     kubectx kind-preprod

     ```

     2.2 Create environment Namespaces (qa and prod)

     ```
     $ kubectl create namespace qa
     $ kubectl create namespace sectest
     $ kubectl get namespaces

     ```

     3.0 Deploy curl to the qa and sectest Namespaces


     ```
     kubectl apply -f - <<EOF
     apiVersion: v1
     kind: Pod
     metadata:
       name: curl-pod
       namespace: qa
     spec:
       containers:
       - name: curlpod
         image: radial/busyboxplus:curl
         command:
         - sh
         - -c
         - while true; do sleep 1; done  
     ---
     apiVersion: v1
     kind: Pod
     metadata:
       name: curl-pod
       namespace: sectest
     spec:
       containers:
       - name: curlpod
         image: radial/busyboxplus:curl
         command:
         - sh
         - -c
         - while true; do sleep 1; done  
     EOF

     ```


     4.0  Deploy an NGNIXwebserver in QA

     4.1 Deploy server
    ```
     kubectl apply -n qa -f - <<EOF
     apiVersion: v1
     kind: Pod
     metadata:
       name: web
     spec:
       containers:
       - image: nginx
         imagePullPolicy: Always
         name: nginx
         ports:
         - containerPort: 80
           protocol: TCP
     EOF
    ```


     4.2 Get the IP address of the web pod in the prod namespace 

     ```
     $ kubectl describe pod web -n qa | grep IP 
     $ export WEB_IP_QA = <IP from above>
     ```

     5.0 try to curl from both qa and sectest 

     ```
     $ kubectl -n sectest  exec curl-pod -- curl --max-time 5.5 -I $WEB_IP_QA
     $ kubectl -n qa  exec curl-pod -- curl --max-time 5.5 -I $WEB_IP_QA

     ```

     6.0 Apply a network policy to prevent all pods outside qa from accessing qa


     ```
     kubectl apply -n qa -f - <<EOF
     apiVersion: networking.k8s.io/v1
     kind: NetworkPolicy
     metadata:
       namespace: qa              
       name: block-other-namespace
     spec:
       podSelector: {}               
       ingress:
       - from:
         - podSelector: {}     
     EOF
     ```



     7.0 Try to curl NGNIX webserver from each environment (sectest fails and qa passes)

     ```
     $ kubectl -n sectest  exec curl-pod -- curl --max-time 5.5 -I $WEB_IP
     $ kubectl -n qa  exec curl-pod -- curl --max-time 5.5 -I $WEB_IP

     ```

#### Task 2 - Prod and Stage
1. Switch to Prod Cluster and Install Calico 
2. Install Kyverno and apply default deny-all policy to all namespaces created 
3. Set-up environment as above (using stage and prod namespace)
4. Try to curl from both stage and prod (should both fail)
5. Create network policies to permit accessing web server in prod from stage
6. Try to curl from both stage and prod (stage fails and prod works)
7. Destroy environment 

??? solve "Solution Task 2"


     1.0  Switch to Prod Cluster and Install Calico 

     1.1 - Make sure we are on the prod cluster

     ```
     kubectx kind-prod
     ```

     1.2 install the Tigera operator on the cluster.

     ```
     $ kubectl create -f https://docs.projectcalico.org/manifests/tigera-operator.yaml
     ```

     1.3 Check calico
     ```
     kubectl get pod -n calico-system
     ```

     1.4 Apply the configuration

     ```
     kubectl apply -f - <<EOF
     apiVersion: operator.tigera.io/v1
     kind: Installation
     metadata:
       name: default
     spec:
       calicoNetwork:
         ipPools:
           - blockSize: 26
             cidr: 10.241.0.0/16
             encapsulation: VXLANCrossSubnet
             natOutgoing: Enabled
             nodeSelector: all()
     EOF
     ```


    2.0 Install Kyverno and apply dedault deny-all policy to all namespaces created 

    2.1 Install Kyverno (Note this is a cluster wide policy )

    ```
     helm install kyverno kyverno/kyverno -n kyverno --create-namespace
    ```

     2.2 apply kyverno policy
     ```
     kubectl apply  -f - <<EOF
     apiVersion: kyverno.io/v1
     kind: ClusterPolicy
     metadata:
       name: add-networkpolicy
       annotations:
         policies.kyverno.io/title: Add Network Policy
         policies.kyverno.io/category: Multi-Tenancy, EKS Best Practices
         policies.kyverno.io/subject: NetworkPolicy
         policies.kyverno.io/minversion: 1.6.0
         policies.kyverno.io/description: >-
           By default, Kubernetes allows communications across all Pods within a cluster.
           The NetworkPolicy resource and a CNI plug-in that supports NetworkPolicy must be used to restrict
           communications. A default NetworkPolicy should be configured for each Namespace to
           default deny all ingress and egress traffic to the Pods in the Namespace. Application
           teams can then configure additional NetworkPolicy resources to allow desired traffic
           to application Pods from select sources. This policy will create a new NetworkPolicy resource
           named `default-deny` which will deny all traffic anytime a new Namespace is created.      
     spec:
       rules:
       - name: default-deny
         match:
           any:
           - resources:
               kinds:
               - Namespace
         generate:
           apiVersion: networking.k8s.io/v1
           kind: NetworkPolicy
           name: default-deny
           namespace: "{{request.object.metadata.name}}"
           synchronize: true
           data:
             spec:
               # select all pods in the namespace
               podSelector: {}
               # deny all traffic
               policyTypes:
               - Ingress
               - Egress
     EOF
     ```

     3.0  Set-up environment as above (using stage and prod namespace)

     3.1 Make e we are on the prepod cluster 

     ```
     kubectx kind-prod
     ```

     3.2 Create environment Namespaces (qa and prod)

     ```
     $ kubectl create namespace stage
     $ kubectl create namespace prod
     $ kubectl get namespaces

     ```

     3.3 Deploy curl to the qa and sectest Namespaces


     ```
     kubectl apply -f - <<EOF
     apiVersion: v1
     kind: Pod
     metadata:
       name: curl-pod
       namespace: stage
     spec:
       containers:
       - name: curlpod
         image: radial/busyboxplus:curl
         command:
         - sh
         - -c
         - while true; do sleep 1; done  
     ---
     apiVersion: v1
     kind: Pod
     metadata:
       name: curl-pod
       namespace: prod
     spec:
       containers:
       - name: curlpod
         image: radial/busyboxplus:curl
         command:
         - sh
         - -c
         - while true; do sleep 1; done  
     EOF

     ```


     3.4  Deploy an NGNIX webserver in prod

    ```
     kubectl apply -n prod -f - <<EOF
     apiVersion: v1
     kind: Pod
     metadata:
       name: web
     spec:
       containers:
       - image: nginx
         imagePullPolicy: Always
         name: nginx
         ports:
         - containerPort: 80
           protocol: TCP
     EOF
    ```


     4.0 Try to curl from both stage and prod (should both fail)

     4.1 Get Web IP

     ```
     $ kubectl describe pod web -n prod | grep IP 
     $ export WEB_IP_PROD = <IP from above>

     ```

     4.2 try to curl from both qa and sectest 
    ```
     $ kubectl -n stage  exec curl-pod -- curl --max-time 5.5 -I $WEB_IP_PROD
     $ kubectl -n prod  exec curl-pod -- curl --max-time 5.5 -I $WEB_IP_PROD
    ```

     5.0 Create network policies to permit accessing web server in prod from prod

    ```
     kubectl apply  -f - <<EOF
     apiVersion: networking.k8s.io/v1
     kind: NetworkPolicy
     metadata:
       namespace: prod              
       name: allow-ingress-prod
     spec:
       podSelector: {}               
       ingress:
       - from:
         - podSelector: {} 
     ---
     apiVersion: networking.k8s.io/v1
     kind: NetworkPolicy
     metadata:
       namespace: prod  
       name: allow-all-egress
     spec:
       podSelector: {}
       egress:
       - {}
       policyTypes:
       - Egress
     EOF

    ```

    6.0 Try to curl from both stage and prod (stage fails and prod works)

    6.1.1 Get IP Address
     ```
     $ kubectl describe pod web -n prod | grep IP 
     $ export WEB_IP = <IP from above>
     ```

    6.2  from both qa and sectest 
    ```
     $ kubectl -n stage  exec curl-pod -- curl --max-time 5.5 -I $WEB_IP_PROD
     $ kubectl -n prod  exec curl-pod -- curl --max-time 5.5 -I $WEB_IP_PROD
    ```

    7.0 Destroy environment 

     ```

     $ Kubectx prod
     $ kubectl delete namespace prod
     $ Kubectl delete namespace stage
     $ Kind delete clusters prod 

     $ Kubectx preprod
     $ kubectl delete namespace qa
     $ kubectl delete namespace sectest


     $ kind delete clusters --all  
     ```

---------------------------------------------------------------------------------------------------

# Additional Challenges
Work in progress...


