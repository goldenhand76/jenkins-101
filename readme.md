
## YouTube Link
For the full 1 hour course watch on youtube:
https://www.youtube.com/watch?v=6YZvp2GwT0A

# Installation
## Build the Jenkins BlueOcean Docker Image (or pull and use the one I built)
```bash
docker build -t myjenkins-blueocean:2.414.2 .

#IF you are having problems building the image yourself, you can pull from my registry (It is version 2.332.3-1 though, the original from the video)

docker pull devopsjourney1/jenkins-blueocean:2.332.3-1 && docker tag devopsjourney1/jenkins-blueocean:2.332.3-1 myjenkins-blueocean:2.332.3-1
```

## Create the network 'jenkins'
```bash
docker network create jenkins
```

## Run the Container
### MacOS / Linux
```bash
docker run --name jenkins-blueocean --restart=on-failure --detach ^
  --network jenkins --env DOCKER_HOST=tcp://docker:2376 ^
  --env DOCKER_CERT_PATH=/certs/client --env DOCKER_TLS_VERIFY=1 ^
  --volume jenkins-data:/var/jenkins_home ^
  --volume jenkins-docker-certs:/certs/client:ro ^
  --publish 8080:8080 --publish 50000:50000 goldenhand/myjenkins:2.440.2-1
```

### Windows
```bash
docker run --name jenkins-blueocean --restart=on-failure --detach `
  --network jenkins --env DOCKER_HOST=tcp://docker:2376 `
  --env DOCKER_CERT_PATH=/certs/client --env DOCKER_TLS_VERIFY=1 `
  --volume jenkins-data:/var/jenkins_home `
  --volume jenkins-docker-certs:/certs/client:ro `
  --publish 8080:8080 --publish 50000:50000 myjenkins-blueocean:2.414.2
```


## Get the Password
```bash
docker exec jenkins-blueocean cat /var/jenkins_home/secrets/initialAdminPassword
```

## Connect to the Jenkins
https://localhost:8080/

## Installation Reference:
https://www.jenkins.io/doc/book/installing/docker/


## alpine/socat container to forward traffic from Jenkins to Docker Desktop on Host Machine

https://stackoverflow.com/questions/47709208/how-to-find-docker-host-uri-to-be-used-in-jenkins-docker-plugin
```bash
docker run -d --restart=always -p 127.0.0.1:2376:2375 --network jenkins -v /var/run/docker.sock:/var/run/docker.sock alpine/socat tcp-listen:2375,fork,reuseaddr unix-connect:/var/run/docker.sock
docker inspect <container_id> | grep IPAddress
```

## Using my Jenkins Python Agent
```bash
docker pull devopsjourney1/myjenkinsagents:python
```

# Update

First identify your image.
```bash
docker ps --format "{{.ID}}: {{.Image}} {{.Names}}"
3d2fb2ab2ca5: jenkins-docker jenkins-docker_1
```

Then login into the image as root.
```bash
docker container exec -u 0 -it jenkins-docker_1 /bin/bash
```

Now you are inside the container, download the `jenkins.war` file from the official site like.
## Download using wget
```bash
wget wget http://updates.jenkins-ci.org/download/war/2.176.1/jenkins.war
```
## Download using curl
```bash 
curl -O http://updates.jenkins-ci.org/download/war/2.176.1/jenkins.war
```
## Copy from local
```bash
docker cp jenkins.war jenkins-blueocean:/usr/share/jenkins/
```

Replace the version with the one that fits to you.

The next step is to move that file and replace the oldest one.
```bash
mv ./jenkins.war /usr/share/jenkins/
```

Then change permissions.
```bash
chown jenkins:jenkins /usr/share/jenkins/jenkins.war
```

The last step is to logout from the container and restart it.
```bash
docker restart jenkins-docker_1
```

You can verify that update was successful by access to you Jenkins url.


# Add Agent Using SSH Key

**From the target slave node's console**

Switch to the root user:

```bash 
sudo su
```

Add a jenkins user with the home /var/lib/jenkins (Note: I am keeping my home directory in /var/lib/jenkins):

```bash
useradd -d /var/lib/jenkins jenkins
```

**From the Jenkins Master**

Copy the /var/lib/jenkins/.ssh/id_rsa.pub key from the Jenkins user on the master

**From the target slave node's console**

Create an authorized_keys file for the Jenkins user
```bash
mkdir /var/lib/jenkins
mkdir /var/lib/jenkins/.ssh
touch /var/lib/jenkins/.ssh/authorized_keys
```
**Paste the key from the Jenkins master into the file.**

Create an ssh key pair using the following command.
```bash
ssh-keygen -t ed25519
```

Add the public to authorized_keys file using the following command.
```bash
cat id_ed25519.pub >> ~/.ssh/authorized_keys
```

Now, copy the contents of the private key to the clipboard.
```bash
cat id_ed25519
```

> **Make sure the files have correct owner and permission.**
>
>```bash
> chown -R jenkins /var/lib/jenkins/.ssh
> chmod 600 /var/lib/jenkins/.ssh/authorized_keys
> chmod 700 /var/lib/jenkins/.ssh
>```