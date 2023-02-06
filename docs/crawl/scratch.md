
# 3.Image From Scratch 

## Introduction

This scenario assumes that you have completed the security verification practices in the last exercise set.  If you have not done this, you should ensure you complete these steps and understand the importance of inspecting an image before using it.  For this scenario, we will recreate an image container from scratch and push it to a container repository and then pull it for use locally. 

 There are several reasons why someone might want to build a Docker image from scratch:

??? Harma "Customization" 
    Building a container image from scratch allows you to fully customize the contents of the image to meet your specific needs. You can include only the libraries, dependencies, and applications that you need, and configure the image in exactly the way that you want.   

??? Harma "Size"
    Building a container image from scratch can result in a smaller final image size, as you only include the components that you need and can optimize the image for your specific use case.

??? Harma "Security"
    Building a Container image from scratch allows you to start with a minimal base image and add only the components that you need, which can help to reduce the attack surface and improve security 
    
??? Harma "Control"
    Building a Docker image from scratch gives you complete control over the contents and configuration of the image, which can be useful if you have specific requirements or constraints.

??? Harma "Learning"
    Building a Docker image from scratch can also be a useful learning exercise, as it allows you to understand the process of creating a Docker image and the various components that go into it. 


#### Single vs multi stage image building

We will also explore building single-stage and multistage container images.  A single-stage container build is a process in which all the necessary steps to create a container image are performed in a single build stage.  This can include building the application, installing dependencies, and packaging the final image.  On the other hand, a multistage container build involves creating the container image in multiple stages.  Each stage in the build process includes only the packages and libraries necessary for that stage rather than the entire build environment.  The final stage of the build process creates the final container image, which only includes the packages and libraries needed for the application.

There are several reasons why it can be beneficial to build containers in multiple stages:

??? Harma "Smaller image size"
    Building a container in multiple stages allows you to create smaller final images. This is because each stage in the build process only includes the packages and libraries necessary for that stage, rather than the entire build environment. This can make it easier to deploy and manage the container, as well as reduce the risk of security vulnerabilities.

??? Harma "Improved Security"
    Building a container in multiple stages can improve security by reducing the attack surface of the final image. This is because the final image only includes the packages and libraries necessary for the application, rather than the entire build environment.

??? Harma "Simplified maintenance"
    Building a container in multiple stages can make it easier to maintain the container over time. This is because you can update or rebuild individual stages of the build process without affecting the entire container.

??? Harma "Improved build efficiency"
    Building a container in multiple stages can improve build efficiency by allowing you to reuse common build steps across multiple containers. This can save time and reduce the complexity of the build process.

Overall, building containers in multiple stages can provide a number of benefits, including smaller image size, improved security, simplified maintenance, and improved build efficiency.

------------------------------------------------------------------------------------

## Supplementary Learning Material  

<iframe width="1120" height="630" src="https://www.youtube.com/embed/JofsaZ3H1qM" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

#### Additional Links:

* Podman for Devops Chapter 6 - <https://learning.oreilly.com/library/view/podman-for-devops/9781803248233/B17908_06_epub.xhtml#_idParaDest-131>
* Buildah as a cli tool - <https://hackernoon.com/>

* Getting started Buildah - <(https://opensource.com/article/18/6/getting-started-buildah>
* Buildah and OCI images - <https://www.linode.com/docs/guides/using-buildah-oci-images/>
* Github buildah tutorial - <https://github.com/containers/buildah/blob/main/docs/tutorials/01-intro.md)> 
* Docker multi-stage - <https://docs.docker.com/build/building/multi-stage>  
* Buildah vs Kaniko - <https://earthly.dev/blog/docker-vs-buildah-vs-kaniko/>
* Distroless - <https://github.com/GoogleContainerTools/distroless>
* Distroless - <https://medium.com/@luke_perry_dev/dockerizing-with-distroless-f3b84ae10f3a>

-----------------------------------------------------------------------------------

## Scenario

1. Install and test Student API Application 
2. Create a Basic Dockerfile
3. Create a multistage DockerFile 
4. Create a single stage Buildah File
5. Creat a multiStage Buildah File
6. Push image to Dockerhub 


**Solutions**

??? solve "Solution"

      1.Install and test Student API Application

      1.1 Set-up environment
      ```
      $ mkdir student
      $ cd student
      $ sudo apt-get install python3-venv 
      $ python3 -m venv venv 
      $ source venv/bin/activate 
      $ pip install fastapi uvicorn 

      # Install tooling to make working with JSON and YAML files easier 
      $ sudo apt-get install jq
      $ wget https://github.com/mikefarah/yq/releases/download/${VERSION}/${BINARY} -O /usr/bin/yq &&\
      chmod +x /usr/bin/yq

      ```

      1.2 Create application

      ```
      cat <<'EOF' >>app.py

      from fastapi import FastAPI, HTTPException
      from typing import Optional
      from pydantic import BaseModel

      app = FastAPI()

      students = [
          {'name': 'Student 1', 'age': 20},
          {'name': 'Student 2', 'age': 18},
          {'name': 'Student 3', 'age': 16}
      ]

      class Student(BaseModel):
          name: str
          age: int

      @app.get('/students')
      def user_list(min: Optional[int] = None, max: Optional[int] = None):

          if min and max:

              filtered_students = list(
                  filter(lambda student: max >= student['age'] >= min, students)
              )

              return {'students': filtered_students}

          return {'students': students}

      @app.get('/students/{student_id}')
      def user_detail(student_id: int):
          student_check(student_id)
          return {'student': students[student_id]}

      @app.post('/students')
      def user_add(student: Student):
          students.append(student)

          return {'student': students[-1]}

      @app.put('/students/{student_id}')
      def user_update(student: Student, student_id: int):
          student_check(student_id)
          students[student_id].update(student)

          return {'student': students[student_id]}

      @app.delete('/students/{student_id}')
      def user_delete(student_id: int):
          student_check(student_id)
          del students[student_id]

          return {'students': students}

      def student_check(student_id):
          if not students[student_id]:
              raise HTTPException(status_code=404, detail='Student Not Found')
      EOF
      ```

      1.3 Run application
      ```
      uvicorn --port 8090 app:app 

      # Create the requirements.txt for next stages 
      $ pip freeze > requirements.txt

      ```

      1.4 Test the API

      ```

      # Get all students
      $ curl http://127.0.0.1:8090/students

      # Filter students based on age
      $ curl "http://127.0.0.1:8090/students?min=16&max=18"

      # Get single student
      $ curl http://127.0.0.1:8090/students/0T
      # Add student
      $ curl -X 'POST' http://127.0.0.1:8090/students -H 'Content-Type: application/json' -d '{"name":"Tekton Operator", "age": 99}'

      # update student
      $ curl -X 'PUT' http://127.0.0.1:8090/students/0 -H 'Content-Type: application/json' -d '{"name":"Student X", "age": 18}'

      # Delete Student 
      $curl -X 'DELETE' http://127.0.0.1:8090/students/3
      ```

      2.0 Create a basic Dockerfile 

      ```
      cat <<'EOF' >>Dockerfile
      FROM python:3.8

      # Install dependencies
      COPY requirements.txt .


      RUN pip install -r requirements.txt 

      # Copy the application code
      COPY . .

      # Run the application
      CMD ["uvicorn", "app:app", "--host", "0.0.0.0", "--port", "8090"]
      EOF
      ```

      2.1 Build and run application

      ```
      $ docker build -t studentbook .
      $ docker run -p 8090:8090 --name sb studentbook

      ```

      3.0 Create a multistage DockerFile 

      ```
      cat <<'EOF' >>Dockerfile
      # Stage 1 - Build stage
      FROM python:3.10-slim AS build-env

      # Copy the application code
      COPY . /app
      WORKDIR /app

      # Install dependencies 
      RUN pip install --no-cache-dir --upgrade -r requirements.txt && cp $(which uvicorn) /app

      # Stage 2 - Distroless container 
      FROM gcr.io/distroless/python3

      # Copy the built application from the build stage
      COPY --from=build-env /app /app
      COPY --from=build-env /usr/local/lib/python3.10/site-packages /usr/local/lib/python3.10/site-packages
      ENV PYTHONPATH=/usr/local/lib/python3.10/site-packages

      WORKDIR /app

      # Run the application 
      CMD ["./uvicorn", "app:app", "--host", "0.0.0.0", "--port", "8090"]
      EOF
      ```

      3.1 Build and run application

      ```
      $ docker build -t studentbook .
      $ docker run -p 8090:8090 studentbook

      ```

      4.0 Create a single stage Buildah File 

      ```
      cat <<'EOF' >>single-buildah.sh
      #!/usr/bin/env bash

      set -euo pipefail

      # Create a new container from the python:3.8-slim image
      container=$(buildah from python:3.8-slim)

      # Install dependencies
      buildah run "$container" -- pip install fastapi uvicorn

      # Copy the application code
      buildah copy "$container" . /app

      # Set the working directory
      buildah config --workingdir /app "$container"

      # Set the entrypoint
      buildah config --entrypoint "uvicorn app:app --host 0.0.0.0 --port 8090" "$container"

      # Commit the container to an image
      buildah commit "$container" studentbook

      # Remove the container
      buildah rm "$container"
      EOF

      sudo chmod +x single-buildah.sh
      ./single-buildah.sh

      $ podman run -p 8090:8090 studentbook

      ```
    

      6.0 Push student book to DockerHub

      # assumes you have DockerHub account set-up

      6.1 Enter docker username and password (NOTE not secure to do it this way)
      ```
      $ docker login -u your_user_name
      ```

      6.2 Tag image (if not already done from above)
      ```
      $ docker build -t studentbook
      $ docker image tag studentbook USER/studentbook:v1.0
      ```

      6.3 Push Image
      ```
      $ docker image push USER/studentbook:v1.0
      $ docker push 
      ```

----------------------------------------------------------------------------

## Additional Challenges

1. **Security tooling container** - Building your own security tooling container can be useful.  Using [Hacker Container](https://github.com/madhuakula/hacker-container) as inspiration, craft your own security tooling container that you can use for future endeavors.  Some design consideration are:
    * Conatiner should be built using a multistage process
    * Keep the container as lean as possible
    * Consider implementing a flag that will only add tools for a given a situation.  For example, you could have a light, medium, and heavy option that includes more or less tooling depending on the flag. 

2. **Create a multistage Buildah file** - Using Buildah, create a multistage build file and deploy the studentbook app in a container. 
   



