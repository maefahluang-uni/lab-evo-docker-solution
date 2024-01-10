# Build Docker on Cloud Infrastructure

## GCP Cloud Shell
Go to GCP console and open cloud shell with the icon on the top right corner. This cloud shell is a virtual machine that already has the gcloud command line tool installed. We will use this tool to interact with GCP.

## Clone your repository
- In the cloud shell, clone your repository by typing:

```bash
git clone [url to your repository]
```

## Setting up repository
We will use Artefact registry on GCP to store our docker images.We can create the repository by going to the console and selecting the project. Then go to the artefact registry and create a repository. We can also create the repository using the gcloud command line tool. 

```bash
gcloud artifacts repositories create docker-repo --repository-format=docker \
    --location=us-central1 --description="Docker repository"
```
This command creates a docker repository in the us-central1 region called "docker-repo". 

## Build for ExpressJS
this is a simple nodejs application that returns "Hello World" when you send a request to it. We will containerize this application using docker and push it to the repository we created in the previous step.
### Config Dockerfile for ExpressJS
- In expressjs folder, edit Dockerfile as the following:

```dockerfile
FROM node:7.7-alpine

# install deps
ADD package.json /tmp/package.json
RUN cd /tmp && npm install

# Copy deps
RUN mkdir -p /opt/hellojs-app && cp -a /tmp/node_modules /opt/hellojs

# Setup workdir
WORKDIR /opt/hellojs-app
COPY . /opt/hellojs-app

# run
EXPOSE 3000
CMD ["npm", "start"]
```
This Dockerfile is used to containerize a Node.js application using the Alpine Linux image. Here's a brief explanation of each section:

    FROM node:7.7-alpine:
        Specifies the base image for the Docker container. In this case, it's the Node.js 7.7 version with the Alpine Linux distribution, which is known for its small size.

    ADD package.json /tmp/package.json:
        Copies the package.json file from the local directory to /tmp/package.json inside the container.

    RUN cd /tmp && npm install:
        Changes the working directory to /tmp and installs the Node.js dependencies specified in package.json.

    RUN mkdir -p /opt/hellojs-app && cp -a /tmp/node_modules /opt/hellojs:
        Creates the directory /opt/hellojs-app inside the container and copies the Node.js dependencies installed in the previous step (/tmp/node_modules) to /opt/hellojs.

    WORKDIR /opt/hellojs-app:
        Sets the working directory inside the container to /opt/hellojs-app.

    COPY . /opt/hellojs-app:
        Copies the remaining application files from the local directory to /opt/hellojs-app inside the container.

    EXPOSE 3000:
        Informs Docker that the application will use port 3000.

    CMD ["npm", "start"]:
        Specifies the default command to run when the container starts. In this case, it runs the npm start command, which typically starts the Node.js application.
        
### Build and push image for ExpressJS
- Go to expressjs folder and run the following command to install node modules:

```bash
npm install
```
- In  expressjs folder, open a terminal and run:

```
docker build -t hellojs .
```
- You can run the image locallly by typing:

```
docker run -p 3000:3000 hellojs
```
The -p 3000:3000 maps port 3000 on the host to port 3000 on the container.

-test this request by typing "curl http://localhost:3000", this should give you a response "Hello World"

- Push the image to the repository, ***please replace [project-id] with your project id on GCP***:

```bash
gcloud submit --region=us-central1 --tag us-central1-docker.pkg.dev/[project-id]/docker-repo/hellojs:latest
```

After the fetching and pushing is done, you can go to the console and check the repository to see the image.

## Build for Spring boot
This is a simple spring boot application that returns "Hello World" when you send a request to it. We will containerize this application using docker and push it to the repository we created in the previous step.
### Config Dockerfile for Spring boot
- In springboot folder, edit Dockerfile as the following:

```dockerfile
FROM eclipse-temurin:17-jdk-alpine
VOLUME /tmp
COPY target/*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java","-jar","/app.jar"]
```
This Dockerfile is used to create a Docker image for a Java application. Here's a brief explanation of each line:

    FROM eclipse-temurin:17-jdk-alpine:
        Specifies the base image for the Docker container. In this case, it uses the Eclipse Temurin (formerly AdoptOpenJDK) 17 JDK with the Alpine Linux distribution, known for its small size.

    VOLUME /tmp:
        Creates a volume named /tmp. Volumes in Docker are used to share data between the host machine and the container.

    COPY target/*.jar app.jar:
        Copies all the JAR files in the target directory (assuming a Maven project structure) from the local build context into the container, and renames it to app.jar.

    ENTRYPOINT ["java","-jar","/app.jar"]:
        Specifies the default command to run when the container starts. In this case, it runs the Java command to execute the JAR file (/app.jar), making it the entry point for the container.

### Build and push image for Spring boot
- In springboot folder, open a terminal and run:

```
docker build -t hellospring .
```
- Run the image -p 8080:8080 maps port 8080 on the host to port 8080 on the container

```
docker run -p 8080:8080 hellospring
```

- test this by typing command "curl http://localhost:8080" this should give you a response "Hello World"

- Push the image to the repository, ***please replace [project-id] with your project id on GCP***:

```bash
gcloud submit --region=us-central1 --tag us-central1-docker.pkg.dev/[project-id]/docker-repo/hellospring:latest
```

## Run container with Cloud Run
We will use Cloud Run to run our container. Cloud Run is a managed compute platform that enables you to run stateless containers that are invocable via HTTP requests. Cloud Run is serverless: it abstracts away all infrastructure management.

Follow the steps below to run your container on Cloud Run:

- Go to the console and select the project. Then go to Cloud Run and create a service.
- Select the image you want to run and click next (choose the image you put in "docker-repo").
- Give the right container port (3000 for expressjs and 8080 for springboot).
- Click create and wait for the service to be created.
- After the service is created, you can test it by clicking on the url on the top of the page. This should give you a response "Hello World".


