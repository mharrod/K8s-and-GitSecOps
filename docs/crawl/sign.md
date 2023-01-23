# 4.Image Signing

## Introduction

Signing container images is a crucial step in ensuring the authenticity and integrity of the images. Signing the images guarantees that they have not been tampered with or compromised in any way. This is particularly important in production environments where containers are used to run critical applications. Signing images ensures security, compliance, and trust. It also provides transparency, allowing one to see which images have been modified and by whom. Signed images can be helpful for auditing and troubleshooting purposes as well as compliance, security, and regulatory requirements such as PCI-DSS, HIPAA, and SOC2. 

Additionally, it helps to establish trust in the images for organizations that share images with others, such as open-source projects or commercial vendors. Although signing images can help to ensure the authenticity and integrity of the images, it is not a replacement for other security measures such as vulnerability scanning or runtime security.
<br>
<br>
There are several options for signing containers, each with its own advantages and use cases.  The best option depends on the specific use case and requirements, and you should carefully evaluate the options available to find the best solution for your needs.

??? Harma "Signing Options"
    Digital Signatures, such as X.509 certificates, are used to sign the container and verify its authenticity. This method is considered to be one of the most secure options for container signing, as it provides a high level of trust and is widely used in enterprise environments.

    Hash-Based Signatures, such as SHA-256, create a unique signature for the container, which can be verified using the same function. This method is considered to be less secure than digital signatures, but it is still a reliable way to ensure the authenticity of the container image.

    Notary is a centralized service that allows users to sign and verify the authenticity of containers. It uses digital signatures, such as X.509 certificates, to sign the container and verify its authenticity. It also provides a centralized repository of signed containers and allows for the management of multiple signing keys.

    TUF (The Update Framework) is an open-source framework for securing software update systems, which can be used to sign and verify container images. It provides a flexible and secure method for managing the distribution of software updates.

    In-Transit encryption, using transport layer security (TLS) or other encryption method to encrypt the container image during transit. This method provides an additional layer of security for container images in transit, but it does not provide the same level of trust as digital signatures or hash-based signatures.

    CoSign is an open-source tool for signing and verifying container images, it uses hash-based signatures, such as SHA-256, to create a unique signature for the container. The tool allows for the management of multiple signing keys, and provides a command line interface for signing and verifying images. It's important to note that cosign only works with container images in the OCI format, it's not compatible with other container runtimes like Docker.

    GPG (GNU Privacy Guard) is a tool that allows you to sign and verify the authenticity of files. It can be used to sign container images before pushing them to a registry, and then use GPG to verify the signature on the image when it is pulled from the registry. This method provides authenticity of the container image, but it will not provide the same level of security as using a centralized service like Notary, which also allows for the management of multiple signing keys.

    Image signing with GCP, Google Cloud Platform has its own image signing service which allow you to sign container images and validate them on Google Kubernetes Engine (GKE) clusters.

    Image signing with AWS: AWS also provide a service called Elastic Container Registry (ECR) Image Scanning which allows you to scan container images for vulnerabilities and malware.


For our purposes we will use Cosign, but most of the principles introduced are transferable to GPG or other mechanisms. 

## Supplementary Learning Material 

<iframe width="1120" height="630" src="https://www.youtube.com/embed/HLb1Q086u6M" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

* **Install cosign** - <https://docs.sigstore.dev/cosign/installation/>
* **Using cosign** - <https://faun.pub/signing-container-images-using-cosign-7df8f61456ad>
* **Using cosign for image signing** - <https://github.com/sigstore/cosign>


Install yq wget https://github.com/mikefarah/yq/releases/latest/download/yq_linux_amd64 -O /usr/bin/yq &&\
    chmod +x /usr/bin/yq

    https://github.com/mikefarah/yq



## Scenario 

For this sceanrio, we will generate a local signing key and use it to sign images.  of course, in the real world private keys would be stored in a much more secure manner.  

1. Install Local Registry (no Authentication)
2. Sign Image with Cosign and verify signature
3. Sign Image and Verify signature
4. Generate, sign, and attach SBOM

**Solution**

??? solve "Solution"

      1.0 Install local container registry (no authentication)


      1.1 Install/start registry 

      ```
      $ docker run -d -p 5001:5001 --restart always --name registry registry:2

      ```
      1.2 Build and tag image 

      ```
      $ docker build -t studentbook .
      $ docker image tag studentbook localhost:5001/studentbook:1.0
      ```

      1.3 Push image to the local registry 
      ```
      $ docker push localhost:5001/studentbook:1.0
      ```

      1.4 Inspect the image
      ```
      $ skopeo copy --tls-verify=false docker://localhost:5001/studentbook oci-archive:stud.tar
      $ skopeo inspect --tls-verify=false docker://localhost:5001/studentbook

      ```

      1.5 Run image from local registry 

      ```
      $ docker pull localhost:5001/studentbook:1.0
      $ docker run -p 8090
      ```

      2.0 Sign image with cosign
         
      2.1 Install cosign
      ```
      # binary
      $ wget "https://github.com/sigstore/cosign/releases/download/v1.6.0/cosign-linux-amd64"
      $ mv cosign-linux-amd64 /usr/local/bin/cosign
      $ chmod +x /usr/local/bin/cosign

      # rpm
      $ wget "https://github.com/sigstore/cosign/releases/download/v1.6.0/cosign-1.6.0.x86_64.rpm"
      $ rpm -ivh cosign-1.6.0.x86_64.rpm

      # dkpg
      $ wget "https://github.com/sigstore/cosign/releases/download/v1.6.0/cosign_1.6.0_amd64.deb"
      $ dpkg -i cosign_1.6.0_amd64.deb


      ```

      2.2 Generate key pair

      ```
      $ cosign generate-key-pair

      ```

      3.0 Sign image and verify signature

      ```
      $ cosign sign --key cosign.key localhost:5001/studentbook/1.0
      $ cosign triangulate localhost:5001/studentbook:1.0 
      $ cosign verify localhost:5001/studentbook:1.0  

      ```

      3.0. SBOM

      3.1 Generate SBOM with Syft
      ```
      $ syft packages localhost:5001/studentbook:1.0 --scope all-layers -o spdx > sb-image.spdx

      ```

      3.2 Attatch SBOM to container package 

      ```
      $ cosign attach sbom --sbom sb-image.spdx localhost:5001/studentbook:1.0
      ```


## Additional Challenges

1. **Add Authentication** - Add authentication to your local registry 
2. **Install a and use a Vault** - Store and use secrets for cosign from a vault












