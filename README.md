# Docker Scout demo service

A repository containing an application and Dockerfile to demonstrate the use of Docker Scout to analyze and remediate CVEs in a container image.
The application consists of a basic ExpressJS server and uses an intentionally old version of Express and Alpine base image.

## Table of Contents

- [Installing Docker Scout](#getting-started)
- [Enabling Docker Scout](#enable-docker-scout)
- [Analyze image vulnerabilities](#analyze-image-vulnerabilities)
- [Fix application vulnerabilities](#fix-application-vulnerabilities)
- [Integrating with GitHub Action](#integrating-with-github-action)

## Getting Started 

## 1. Inner-Loop (using Docker Desktop)

- Install the latest version of Scout CLI

```
curl -fsSL https://raw.githubusercontent.com/docker/scout-cli/main/install.sh -o install-scout.sh
sh install-scout.sh
```

If you're using Docker Desktop, you can enable background SBOM indexing as shown:

<img width="1206" alt="image" src="https://github.com/user-attachments/assets/e320a227-da85-4714-bed2-9eb5b8f64e68">


## Clone the repo

```
 git clone https://github.com/dockersamples/scout-demo-service
 cd scout-demo-service
```

## Build the image, naming it to match the organization you will push it to, and tag it as “v1”:

```shell
docker build -t scout-demo:v1 .
docker run scout-demo:v1
```

Access the app:

```
curl localhost:3000
Hello World!
```

If you're using Docker Desktop, you should be able to see vulnerabilities right now on your Docker dashboard.

<img width="1176" alt="image" src="https://github.com/user-attachments/assets/d6d8cda2-db13-4512-8b28-4be63f4ebb93">

There are 2 major vulnerabilties reported - the first one is related to OpenSSL package and other one is with Express 4.17.3.
It says that Impact Versions of Express.js prior to 4.19.2 and pre-release alpha and beta versions before 5.0.0-beta.3 are affected by an open redirect vulnerability using malformed URLs. 
That means we need to update our Express v4.17.3 to 4.19.2


<img width="1030" alt="image" src="https://github.com/user-attachments/assets/af409b26-92d5-4cec-812f-e1498a8e9d14">


Alternatively, you can see the list of vulnerabilities locally using your terminal.

```
  docker scout cves scout-demo:v1
```



## Fix application vulnerabilities

The fix suggested by Docker Scout is to update the underlying vulnerable express version to 4.17.3 or later.

Update the package.json file with the new package version.


```
…
"dependencies": {
     "express": "4.19.2"
     …
}
```

```
docker build -t scout-demo:v2 .
```

<img width="1162" alt="image" src="https://github.com/user-attachments/assets/9f3d057a-c917-4aa8-be1c-cbff34d36611">


You will find that express vulnerabilities are now fixed.

<img width="1200" alt="image" src="https://github.com/user-attachments/assets/ee4ab5e5-e855-4bd7-b340-30ef66ffcb62">

You will see that the OpenSSL vulnerability is still there. To fix this, open up your Dockerfile and add openssl as shown below:

```
RUN apk add --no-cache \
  nodejs \
  openssl 
```

Try re-building the Docker image with v3.0 this time:

```
docker build -t scout-demo:v3 .
```

This time, you will find all the vulnerabilities are fixed.

<img width="1490" alt="image" src="https://github.com/user-attachments/assets/d00a7099-8eed-4d40-aa00-24636ab14301">



## 2. Using Docker Hub



## Create and push the Docker image to the Docker Hub repository

```
 docker push <org-name>/scout-demo:v1
```

Alternatively, you can use Docker Dashboard directly to to push your Docker image to the Docker Hub.

<img width="1065" alt="image" src="https://github.com/user-attachments/assets/19934207-3a80-4d44-9f3c-33f5e7b744e0">


## Enable Docker Scout

You can enable Docker image analysis right on your Docker Hub repositories - either through CLI or directly using Docker Hub Dashboard.


<img width="1349" alt="image" src="https://github.com/user-attachments/assets/61708231-402d-4c7a-bc00-220b3c899cb0">


Docker Scout analyzes all local images by default. To analyze images in remote repositories, you need to enable it first. You can do this from Docker Hub, the Docker Scout Dashboard, and CLI. Find out how in the overview guide.

Use the Docker CLI docker scout repo enable command to enable analysis on an existing repository with the following command:


```
 docker scout repo enable <org-name>/scout-demo
```

For Example:


```
 docker scout repo enable <org-name>/scout-demo
    ✓ Enabled Docker Scout on <org-name>/lamp-for-collabnix
    ✓ Enabled Docker Scout on <org-name>/ol7-webdeliverer
    ✓ Enabled Docker Scout on <org-name>/puppet-for-docker
    ✓ Enabled Docker Scout on <org-name>/puppet4docker
    ✓ Enabled Docker Scout on <org-name>/scout-demo
```

## Analyze image vulnerabilities

Click on the tag version to see the list of vulnerabilities:


<img width="1339" alt="image" src="https://github.com/user-attachments/assets/7db07085-e73f-4925-86a0-5ac83301762d">

You can see the similar kind of result as you see locally on your Docker Desktop.


<img width="1329" alt="image" src="https://github.com/user-attachments/assets/8433d2c9-f682-4ee3-8845-72b0f75fffe9">




After building, you can use Docker Desktop or the docker scout CLI command to see vulnerabilities detected by Docker Scout.

Using Docker Desktop, select the image name in the Images view to see the image layer view. In the image hierarchy section, you can see which layers introduce vulnerabilities and the details of those.


```
  docker scout cves <org-name>/scout-demo:v1
```

Now you can follow the above instructions to fix it directly on Docker Desktop.


Docker Scout creates and maintains its vulnerability database by ingesting and collating vulnerability data from multiple sources continuously. These sources include many recognizable package repositories and trusted security trackers. You can find more details in the [Advisory Database sources document](https://docs.docker.com/scout/advisory-db-sources/).



## Integrating with GitHub Action

Clone this repo to your own account.  Go to settings > Secrets and Variables > Actions and add DOCKER_PAT and DOCKER_USER.

<img width="1070" alt="image" src="https://github.com/user-attachments/assets/b32a2381-f426-4067-a21d-b6f8ba95f6eb">


Just modify the Docker Hub registry credentials and add the following secrets under GitHub:

- DOCKER_USER: The username for the Docker registry.
- DOCKER_PAT: The personal access token (PAT) or password for the Docker registry.

Ensure that you have the following entries in your workflow modified:

```
          username: ${{ secrets.DOCKER_USER }}
          password: ${{ secrets.DOCKER_PAT }}
```



Run the GitHub Action job and you will see the following output once the job gets completed.

<img width="1241" alt="image" src="https://github.com/ajeetraina/scout-demo-service/assets/313480/1c5638ef-594e-4b4f-b6ac-7fe9ee839774">

