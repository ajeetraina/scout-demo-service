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

- Install the latest version of Scout CLI

```
curl -fsSL https://raw.githubusercontent.com/docker/scout-cli/main/install.sh -o install-scout.sh
sh install-scout.sh
```

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


## Create and push the repository on Docker Hub:

```
 docker push <org-name>/scout-demo:v1
```

## Enable Docker Scout

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

After building, you can use Docker Desktop or the docker scout CLI command to see vulnerabilities detected by Docker Scout.

Using Docker Desktop, select the image name in the Images view to see the image layer view. In the image hierarchy section, you can see which layers introduce vulnerabilities and the details of those.


```
  docker scout cves <org-name>/scout-demo:v1
```




<img width="1084" alt="image" src="https://github.com/dockersamples/scout-demo-service/assets/313480/35015241-4fb8-4437-a511-7dda74710049">

Select layer 5 to focus on the vulnerability introduced in that layer.

<img width="1076" alt="image" src="https://github.com/dockersamples/scout-demo-service/assets/313480/2ef69e3e-be2c-4f85-8aee-f4ca287a9699">


Toggle the disclosure triangle next to express 4.17.1 and then the CVE ID (in this case, “CVE-2022-24999⁠”) to see details of the vulnerability.

<img width="966" alt="image" src="https://github.com/dockersamples/scout-demo-service/assets/313480/f42eff02-0059-46dd-9226-ca7a01857c42">

<img width="1080" alt="image" src="https://github.com/dockersamples/scout-demo-service/assets/313480/2f86c05b-2f8a-4efe-92d8-ddf9efeae49c">



You can also use the Docker CLI to see the same results.


```
docker scout cves <org-name>/scout-demo:v1
    ✓ Provenance obtained from attestation
    ✓ Image stored for indexing
    ✓ Indexed 79 packages
    ✗ Detected 6 vulnerable packages with a total of 26 vulnerabilities

 ...
...

28 vulnerabilities found in 6 packages
  UNSPECIFIED  1
  LOW          0
  MEDIUM       7
  HIGH         18
  CRITICAL     2
```

Docker Scout creates and maintains its vulnerability database by ingesting and collating vulnerability data from multiple sources continuously. These sources include many recognizable package repositories and trusted security trackers. You can find more details in the [Advisory Database sources document](https://docs.docker.com/scout/advisory-db-sources/).

## Fix application vulnerabilities

The fix suggested by Docker Scout is to update the underlying vulnerable express version to 4.17.3 or later.

Update the package.json file with the new package version.


```
…
"dependencies": {
     "express": "4.17.3"
     …
}
```

## Rebuild the image, giving it a new version tag:

```
docker build -t <org-name>/scout-demo:v2 .
```

## Push the image to the same repository on Docker Hub using a new version tag:


```
 docker push <org-name>/scout-demo:v2
```

Now, viewing the latest tag of the image in Docker Desktop, the Docker Scout Dashboard, or CLI, you can see that you have fixed the vulnerability.

<img width="1025" alt="image" src="https://github.com/dockersamples/scout-demo-service/assets/313480/57d43a70-4e19-4b2c-9279-342c4ad5002e">


## Integrating with GitHub Action

Just modify the Docker Hub registry credentials and add the following secrets under GitHub:

- DOCKER_USER: The username for the Docker registry.
- DOCKER_PAT: The personal access token (PAT) or password for the Docker registry.


<img width="1241" alt="image" src="https://github.com/ajeetraina/scout-demo-service/assets/313480/1c5638ef-594e-4b4f-b6ac-7fe9ee839774">


