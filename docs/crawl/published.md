# 2.Containerized Application

## Introduction

Before one uses a 3rd party image/container, one should ensure that they thoroughly inspect and validate the integrity of the image .  Scanning a container image before using it is crucial because it allows you to identify any known vulnerabilities in the image that attackers could potentially exploit.  These vulnerabilities can exist in the base operating system or in any packages installed in the image.  Scanning the image also allows you to verify that the image has not been tampered with and is what the publisher intended. 

An even better approach is to re-build 3rd party container images using the Dockerfile, rather than just downloading and using an image from an external container registry.  There are several reasons why you might choose to build a container image from a Dockerfile rather than use an existing image from Docker Hub:

??? Harma "Customization"
    Building an image from a Dockerfile allows you to customize the image to include exactly the packages and configurations that you need. This can be useful if you need to add specific packages or make other customizations that are not included in the existing images on Docker Hub.

??? Harma "Version Control"
    By building an image from a Dockerfile, you can store the Dockerfile in version control along with your application code. This allows you to track changes to the image over time and roll back to previous versions if necessary.

??? Harma "Reproducibility"
    Building an image from a Dockerfile allows you to ensure that the image is built consistently every time. This can be useful if you need to deploy the image in multiple environments and want to ensure that the image is the same in each environment.

??? Harma "Security"
    Building an image from a Dockerfile allows you to audit the exact steps that were used to build the image. This can be helpful for security purposes, as you can review the commands in the Dockerfile to ensure that they do not include any vulnerabilities.

Overall, building an image from a Dockerfile can provide you with more control and flexibility when deploying your applications with Docker.


## Supplementary Learning Material

<iframe width="1120" height="630" src="https://www.youtube.com/embed/V9JZauO_YAc" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

Installations:

* Installing Trivy for container introspection - <https://aquasecurity.github.io/trivy/v0.18.3/installation/>
* Installing Skopeo for image interaction - <https://github.com/containers/skopeo/blob/main/install.md>
* Installing Podman for containers - <https://podman.io/getting-started/installation>
* Installing Buildah for building containers from scratch - <https://github.com/containers/buildah/blob/main/install.md)>

Container Security Testing:

* Cli tools for containers - <https://snyk.io/blog/command-line-tools-for-containers/> 
* Testing with Trivy - <https://semaphoreci.com/blog/continuous-container-vulnerability-testing-with-trivy>
* Containers and Semgrep - <https://kondukto.io/blog/docker-security-best-practices-with-semgrep>
* Dockerfile best practices - <https://github.com/kondukto-io/dockerfile-bestpractice-rules>
* Using hadolint - <https://github.com/hadolint/hadolint>
* Hadolint - <https://www.containiq.com/post/hadolint>  
* Syft and Grype - <https://medium.com/rahasak/container-vulnerability-scan-with-syft-and-grype-f4ec9cd4d7f1>
* Trivy <https://medium.com/ascentic-technology/secure-container-images-with-trivy-1ef12b5b9b4d>


## Scenario 

1. Inspect Image on GitHub Repo with Trivy
2. Inspect the Dockerfile with Hadolint
3. Check for Secrets with gggshield
4. Inspect with Skopeo on DockerHub
5. Download Image as OCI Archive and scan with Grype 
6. Run Software Composite Analysis with Syft
7. Pull Juiceshop image and rerun security testing
8. Check for misconfigurations with DockerBench  
9. Run container and scan with NMAP and Nikto 
 

**Solution** 

??? solve "Solution"

      1.Inspect Image on GitHub Repo with Trivy - Look for security problems BEFORE you download one bit of the image.

      ```
      $ trivy repo https://github.com/juice-shop/juice-shop | tee /tmp/juice/outputs/trivy.txt

      #Note try this to see what it would look like if results were available 
      $ trivy repo https://github.com/knqyf263/trivy-ci-test

      ```

      2.Inspect the Dockerfile with Hadolint - Easy way to catch some misconfigurations before you download an image from DockerHub and/or rebuild from a Dockerfile 

      ```
      # Clone repo
      $ git clone https://github.com/juice-shop/juice-shop.git

      # Download and install Hadolint
      $ docker pull ghcr.io/hadolint/hadolint  

      # Run Hadolint
      $ docker run --rm -i ghcr.io/hadolint/hadolint < Dockerfile  | tee /tmp/juice/outputs/hadolint.txt
      ```

      3.Check for Secrets with gggshield - Note: This particular tool is not opensource but free for low use scenarios.  You will need to sign-up before using.
      ```
      $ export GITGUARDIAN_API_KEY=the-token-you-got-from-dashboard
      $ python3 -m venv /tmp/juice/venv
      $ /tmp//juice/venv/bin/pip install ggshield
      $ ggshield scan docker bad-secrets
      ```

      4.Inspect with Skopeo on DockerHub - Again before downloading, look for integrity issues before downloading. 

      ```
      $ skopeo inspect docker://bkimminich/juice-shop | jq '.' | tee /tmp/juice/outputs/skopeo.txt

      # To just see the config
      $ skopeo inspect --config docker://bkimminich/juice-shop | jq '.'   

      # list just the tags
      skopeo list-tags docker://bkimminich/juice-shop


      ```

      5.Download Image as OCI Archive and scan with Grype - Downloading as an archived TAR to a isolated location. Not pulling the image just yet. We want to check for known vulnerabilities. 

      ```

      # Copy image to local OCI Archive
      $ skopeo copy docker://bkimminich/juice-shop oci-archive:/tmp/juice/juice-shop.tar 

      # Install Grype
      $ curl -sSfL https://raw.githubusercontent.com/anchore/grype/main/install.sh | sudo sh -s -- -b /usr/local/bin

      # Scan with Grype 
      $ grype oci-archive:/tmp/juice/juice-shop.tar --scope all-layers | tee /tmp/juice/outputs/grype.txt

      # Bonus Snyk has great tooling to do this.  You will need to set-up a free tier account.
      $ snyk container test oci-archive:/tmp/juice/juice-shop.tar 

      ```

      6.Run Software Composite Analysis with Syft - Output a SBOM for use in upstream process and to see what dependencies are in play. 

      ```
      # Install Syft
      $ curl -sSfL https://raw.githubusercontent.com/anchore/syft/main/install.sh | sudo sh -s -- -b /usr/local/bin

      # Generate SBOM from Image
      $ syft packages docker.io/bkimminich/juice-shop --scope all-layers  -o json

      # Generate SBOM from Archieve 
      $ syft packages oci-archive:/tmp/juice/juice-shop.tar --scope all-layers  -o json | tee /tmp/juice/outputs/syft.txt


      ```

      7.Pull Juiceshop image and rerun security testing - Image has been thoroughly tested even before downloading.  Now we can download it and run testing again.  This is overkill but shows that most of the testing can also be run against the local image, not just an archive.  

      ```

      $ docker pull bkimminich/juice-shop 

      # Scan with Grype
      $ grype docker.io/bkimminich/juice-shop --scope all-layers 
      # Scan with Trivy
      $ trivy image bkimminich/juice-shop

      # Run Syft
      $ syft packages docker.io/bkimminich/juice-shop --scope all-layers  -o json

      ```


      8.Check for misconfigurations with DockerBench - Do this before running the image.

      ```
      $ cd /tmp/juice
      $ git clone https://github.com/docker/docker-bench-security.git
      $ cd docker-bench-security

      $ ./docker-bench-security.sh -i juice-shop | tee /tmp/juice/outputs/bench.txt

      ```

      9.Run container and scan with NMAP and Nikto - There are much better commercial and free scanners to use but this is a good starting point to see how DAST can be incorporated early on in the pipeline. 

      ```
      # Run the docker container

      $ docker run --rm -p 3000:3000 bkimminich/juice-shop

      # Get tooling
      docker pull instrumentisto/nmap 
      docker pull securecodebox/nmap
      docker pull frapsoft/nikto     

      # Get Container IP Address (when running locally)
      # Docker ps #get container ID
      docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' container_name_or_id   

      # Scan with NMAp
      $ docker run --rm -it instrumentisto/nmap -A -T4 172.17.0.3  -p 3000  | tee /tmp/juice/outputs/nmap.txt
      $ docker run --rm -it frapsoft/nikto -h 172.17.0.3:3000 


      ```


## Additional Challenges

1. **Automate** - Semi-automate your 3rd party container image security assessment process with a Bash script 
2. **Better Tooling** - Add some bigger and broader tooling to the security verification pipeline. 
    * ADD OWASP Zap as an additional web scanning testing tool. <https://github.com/zaproxy/zaproxy>
    * Implement OPENVas alongside Nessus. <https://github.com/greenbone/openvas-scanner>
    * Include Snyk Scanning for software composite analysis as part of the workflow. <https://snyk.io/>






