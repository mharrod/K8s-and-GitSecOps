# 2.Build Images with Kaniko

## Introduction

Now that we have built a container from scratch and provisioned Kubernetes, we can deploy our application to the Kubernetes cluster using Kaniko. Kaniko is an open-source tool that allows building container images from a Dockerfile inside a container or Kubernetes cluster without needing a Docker daemon. This allows for building container images in a secure and isolated environment and eliminates the need to install or configure a Docker daemon on the build machine.

Kaniko uses a Google-developed library called "containerd" to interact with the host filesystem and build the container images.  Kaniko is particularly useful in scenarios where a Docker daemon cannot be used, such as building images in a Kubernetes cluster or on a machine that doesn’t have Docker installed. Kaniko also allows for building images in a secure environment, as the build process is isolated from the host system and does not require access to the host's Docker daemon.

Kaniko is a tool that is well-suited for use in the context of Kubernetes for several reasons:

??? Harma  "Can build containers in a container"
    Kaniko is designed to build container images inside a container, rather than on the host system. This makes it suitable for use in environments where the host system does not have access to the container engine, such as in a Kubernetes cluster.


??? Harma  "Secure"
    Kaniko uses a number of security features to ensure that the container images it builds are secure. These features include user namespaces, image signing, and image scanning.


??? Harma  "Fast"
    Kaniko is designed to be fast and efficient, with a focus on minimizing the time it takes to build container images. This can be particularly beneficial in the context of Kubernetes, where fast deployment times are often a priority.


??? Harma  "Easy to use:"
    Kaniko is easy to use and integrates well with a variety of tools and environments, including Kubernetes.

----------------------------------------------------------------------------------------

## Supplementary Learning

<iframe width="1120" height="630" src="https://www.youtube.com/embed/EgwVQN6GNJg" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

Additional Links:

* Kind - <https://kind.sigs.k8s.io/>
* Kubernetes - <https://kubernetes.io/>
* Kubernetes with Kind - <https://www.baeldung.com/ops/kubernetes-kind>
* Kaniko and Docker- <https://www.devopsmadness.com/kaniko_build_docker_images>
* Kaniko and Kubernetes - <https://computingforgeeks.com/build-container-images-using-kaniko-in-kubernetes/>
* Kaniko and container tools(local build context) - <https://github.com/GoogleContainerTools/kaniko/blob/main/docs/tutorial.md>


## Scenario Steps
You will need to fork and clone the [StudentBook](https://github.com/mharrod/StudentBook.git) repository 

1. Set-up Kubernetes cluster with Kind
2. Configure Kaniko Git build context 
3. Create the Kaniko Pod Manifest and apply to Kubernetes
4. Pull and Test the image in Docker
5. Pull and Test the image in Kubernetes
6. Rinse and repeat but wih a local Kaniko build context (file)

----------------------------------------------------------------------

## Suggested Solution

??? solve "Solution"

      1.0  Set-up Kubernetes Cluster with Kind

      1.1 Install Kind

      ```
      curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.17.0/kind-linux-amd64
      chmod +x ./kind
      sudo mv ./kind /usr/local/bin/kind

      # If you are going to use podmane then you need to add it as an external provider in your ~/.bashrc or ~/.zshrc file
      $ export KIND_EXPERIMENTAL_PROVIDER=podman

      # If you like, you can alos add an alias to the docker command to your ~/.bashrc or ~/.zshrc file
      alias docker='podman'

      # Other things you may need to do can be found here https://kind.sigs.k8s.io/docs/user/rootless/
      ```
      1.2 Create basic cluster (default name Kind)

      ```
      $ kind create cluster

      ```
      ----------------------------------------------------------------
      2.0 Create the container registry Secret
      <br>
      <br>
      In an ideal scenario, you would be best to use a vault such as hashicorp to store variables.  However, for now, we will just use system environment to store variables and explore our other options at a later date. PLEASE NOTE, secrets don this way are not really secret (not encrypted just encoded)

      2.1 Option 1 

      ```
      $ kubectl create secret docker-registry docker --docker-server=<your-registry-server> --docker-username=<your-name> --docker-password=<your-pword> --docker-email=<your-email>
      ```

      2.2 Option 2 
      ```
      $ cd /tmp
      $ mkdir pass
      $ cd pass

      $ echo -n  'https://index.docker.io/v1/' | base64 > ./server.txt
      $ echo -n 'your-username' > ./username.txt
      $ echo -n 'your-password' > ./password.txt
      $ echo -n 'your-email' > ./email.txt

      $ kubectl create secret generic regcred \
         --from-file=docker-server=./server.txt \
         --from-file=username=./username.txt \
         --from-file=docker-password=./password.txt \
         --from-file=docker-email=./email.txt

      2.3 Delete pass directory if you made it
      $  tmp rm -rf pass
       v
      ```

      2.3 Verify secrets

      ```  
      $ kubectl get secret regcred --output="jsonpath={.data.\.dockerconfigjson}" | base64 --decode
      ```
       

      ----------------------------------------------------------------
      3.0 Create the Kaniko pod manifest and apply to kubernetes

      3.1 Git clone studentbook
      ```
      $ git clone https://github.com/mharrod/StudentBook.git
      ```
      3.2 Apply Kaniko Deployment file 
      ```
      apiVersion: v1
      kind: Pod
      metadata:
        name: kaniko
      spec:
        containers:
        - name: kaniko
          image: gcr.io/kaniko-project/executor:latest
          args: ["--dockerfile=Dockerfile",
                  "--context=git://github.com/mharrod/StudentBook.git",
                  "--destination=mharrod/studentbook-kaniko-repo:1.0"]
          volumeMounts:
          - name: kaniko-secret
            mountPath: "/kaniko/.docker"
        volumes:
        - name: kaniko-secret
          secret:
            secretName: docker
            items:
             - key: .dockerconfigjson
               path: config.json
        restartPolicy: Never
      ```

      3.3 check comnpletion of job
      ```
      $ kubectl logs kaniko -f
      $ kubectl get pods

      ```
      ----------------------------------------------------------------
      4 Pull and test the image in Docker

      ```
      docker run -it mharrod/studentbook-kaniko-repo:1.0
      ```
      ----------------------------------------------------------------
      5.0 Pull and test the image in Kubernetes

      ```
      kubectl apply -f - <<EOF
      apiVersion: apps/v1
      kind: Deployment
      metadata:
        name: studentbook
      spec:
        selector:
          matchLabels:
            app: studentbook 
        replicas: 1
        template:
          metadata:
            labels:
              app: hello
          spec:
            containers:
            - name: studentbook
              image: mharrod/studentbook-kaniko-repo:1.0
      EOF
      ```

      ----------------------------------------------------------------
      6.0 Configure Kaniko local build context 

      This is a less ideal way to do it, but shows how it can work without Github. Make sure you are working in your Studentbook repo that you cloned earlier


      6.1  Prepare local cluster
      ```
      $docker ps
      $docker exec -it [contaniner:id] /bin/sh 
      $ mkdir kaniko && cd kaniko
      $ exit
      $ docker cp Dockerfile container_id:/kaniko/Dockerfile
      ```

      6.2 kubectl create persistent volume**

      ```
      kubectl apply -f - <<EOF
      $ kubectl create -f - <<EOF
      apiVersion: v1
      kind: PersistentVolume
      metadata:
      name: dockerfile
      labels:
      type: local
      spec:
      capacity:
      storage: 10Gi
      accessModes:
      - ReadWriteOnce
      storageClassName: local-storage
      hostPath:
      path: /kaniko # replace with local directory, such as "/home/<user-name>/kaniko"
      EOF
      ```

      6.3 create persistent volume claim

      ```
      $ kubectl create -f - <<EOF
      kind: PersistentVolumeClaim
      apiVersion: v1
      metadata:
      name: dockerfile-claim
      spec:
      accessModes:
         - ReadWriteOnce
      resources:
         requests:
         storage: 8Gi
      storageClassName: local-storage
      EOF
      ```

      6.4 create pod and push to DockerHub

      ```
      $ kubectl create -f - <<EOF
      apiVersion: v1
      kind: Pod
      metadata:
      name: kaniko
      spec:
      containers:
      - name: kaniko
         image: gcr.io/kaniko-project/executor:latest
         args: ["--dockerfile=/workspace/dockerfile",
                 "--context=dir://workspace",
                 "--destination=mharrod/StudentBook-kaniko-local:1.0"]
         volumeMounts:
         - name: kaniko-secret
             mountPath: /kaniko/.docker
         - name: dockerfile-storage
             mountPath: /workspace
      restartPolicy: Never
      volumes:
         - name: kaniko-secret
         secret:
             secretName: regcred
             items:
             - key: .dockerconfigjson
                 path: config.json
         - name: dockerfile-storage
         persistentVolumeClaim:
             claimName: dockerfile-claim
      EOF
      ```

      6.5 Check it worked
      ```
      kubectl get pods
      ```

----------------------------------------------------------------

## Additional Challenges

1. **Try podman/buildah** - Use Podman and Buildah to deploy application to Kubernetes. 
2. **Write a webhook** - Automate the kaniko build when a change/update happens to the dockerfile in the git repo.




