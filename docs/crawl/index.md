# Container Images

## Introduction

For the crawl stage the focus is on first installing applications locally and then securely converting these to image containers using a distroless base container and running them through a battery of security testing and signing the images to prove providence. Distroless containers are a type of container image that do not include a Linux distribution, such as Alpine or Ubuntu. Instead, they only contain the necessary libraries and dependencies to run a specific application. This can lead to smaller image sizes and improved security by reducing the attack surface.
<br>
<br>

```

                        +----------------------+       +----------------------+      +----------------------+
                        |                      |       |Verify the distroless |      |  Build app container |
                        |        GitHub        +------>|container base images +----->|images with distroless|
                        |                      |       | using Cosign before  |      | verfied base images  |
                        |                      |       |       build          |      |                      |
                        +----------------------+       +----------------------+      +----------+-----------+
                                                                                                |
                                                                                                |
                                                                                                |
                                                                                                v
                        +----------------------+      +----------------------+      +----------------------+
                        |                      |      |                      |      |                      |
                        | Verify the container |      |   Cosign the app     |      |   Container image    |
                        |  image using Cosign  |<-----+  container image     |<-----+      scanning        |
                        |                      |      |                      |      |                      |
                        +----------------------+      +----------------------+      +----------------------+

```
<br>

The ways to ensure the integrity of images include but are not limited to:

* **Digital signatures** - Container images can be digitally signed to ensure that they have not been tampered with. By verifying the signature before running the image, you can ensure that the image has not been modified since it was signed.

* **Image scanning** - Tools such as Clair or Aqua Security's MicroScanner can be used to scan container images for known vulnerabilities and other issues. This can help identify any tampering or issues with the image that might compromise its integrity.

* **Image signing and scanning during the build process** - By signing and scanning images as part of the build process, you can ensure that the images are secure and have not been tampered with before they are published.

* **Regular updates** - It is important to regularly update container images to ensure that they include the latest security patches and updates. This can help protect against vulnerabilities that might have been introduced since the image was originally published.


We will be using to simple applications to help us explore these scenarios:

1. [JuiceShop](https://github.com/juice-shop/juice-shop) - OWASP Juice Shop is probably the most modern and sophisticated insecure web application! It can be used in security trainings, awareness demos, CTFs and as a guinea pig for security tools
2. [StudentBook](https://github.com/mharrod/StudentBook) - a small mciroservice based on FastAPI.  simpler than Juiceshop and will give readers a chance to explore some of the intricacies of containerization. 

## Scenarios

* **Local Application** - Download and deploy JuiceShop locally (no containers) and run through the basics of application security.
* **Containerized Application** - Practice validating and verifying a third party image.  The downloading and publishing the application in a container, securely choosing and using a container published by someone else
* **Image from Scratch** - Securely building a container from scratch using Podman, Skopeo, Buildah, and Kaniko
* **Image Integrity** - Validating the integrity of our scratch containers







