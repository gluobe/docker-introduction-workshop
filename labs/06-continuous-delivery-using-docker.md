# Lab 06 - Continuous Delivery using Docker

## Prerequisites

Before procedding with this lab, please create an account on [GitHub](https://github.com/join)

Also, before proceeding with this lab, please familiarize yourself first with the topics below:

* [Continuous Delivery](https://en.wikipedia.org/wiki/Continuous_delivery)

## Forking the repository 

Using a browser surf to https://github.com/gluobe/container-info and click the `Fork` button on the top right of the page.  This should take less than a minute and aftwerwards you will have a fork of the gluobe/container-info repository under you own GitHub account.

## Create an automated build in Docker Hub

* Surf to [https://hub.docker.com/](https://hub.docker.com/)
* Login if not already logged-in
* Click the `Create` button on the top right
* Click the `Create Automated Build` option
* Link your GitHub account if not already linked
* Create an Auto-build for GitHub
* Select the correct user/organization and repository
* Add a description and click the `Create` button

## Start the application

The automated build should be triggered automatically, because we are using a free account you might have to wait a couple of minutes before the build starts.  Once the build complete you can start the conatiner with the following command:

```
docker run -d -p 80:80 --name container-info <DOCKERHUB_USERNAME>/container-info
```

NOTE:
- we are adding the `--name` tag so we can more easily stop and start the container
- it is important that you are starting your own container, so make sure you replace <DOCKERHUB_USERNAME> with your Docker Hub username

## Start a Jenkins Container

Start a Jenkins container using the following command:

```
docker run -d -p 8080:8080 -v /var/run/docker.sock:/var/run/docker.sock -v /usr/bin/docker:/usr/bin/docker -v /usr/lib/x86_64-linux-gnu/libapparmor.so.1.1.0:/lib/x86_64-linux-gnu/libapparmor.so.1 --name jenkins trescst/jenkins
```

NOTE: do not worry to much about the `-v` options, these are required to run Docker inside of Docker (this is out-of-scope for an introduction course)

## Configure Jenkins

Surf to http://nodeXY.PROJECT_NAME.gluo.io:8080, you will see that you need to enter a secret first.  To get the secret you will need to run a command inside the container, this can easily be done using the following command:

```
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

Click the left button to install the most common plugins.  When the installation of the additional plugins is finished create an admin user (use `admin` for both username and password).

## Create a Jenkins job

* Click the `create new job` links
* Enter a name (for example: container-info-deploy) and select `Freestyle project` and click the `OK` button
* Scroll down to the `Build` section
* Click the `Add build step` and select the `execute shell` option
* In this field copy/paste:

```
# stop and remove the container-info container
docker stop container-info
docker rm container-info

# pull the latest container-info image
docker pull trescst/container-info

# start a new container-info container (using the latest image)
docker run -d -p 80:80 --name container-info <DOCKERHUB_USERNAME>/container-info
```
* Scroll back up to the `Build Triggers` section, and select `Trigger builds remotely (e.g., from scripts)`
* Enter a random TOKEN_NAME, for example "ChooseYourOwnToken"
* Click the `Apply` button and then the `Save` button

## Decrease security (for webhook access)

For demo purposes we will allow anonymous access, required for webhook access.  This is, of course, not a recommended setup for production environments.

* Click the Jenkins logo on the top left
* Click the `Manage Jenkins` link
* Click the `Configure Global Security`
* Under `Access Control` and `Authorization` select the `Anyone can do anything` option
* Click the `Apply` button and then the `Save` button