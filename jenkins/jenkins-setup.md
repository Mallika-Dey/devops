# Jenkins Setup

```bash
## Step 1: Create Docker Network

docker network create jenkins

## Step 2: Run Docker-in-Docker (DinD) Container

docker run --name jenkins-docker --rm --detach \
  --privileged --network jenkins --network-alias docker \
  --env DOCKER_TLS_CERTDIR=/certs \
  --volume jenkins-docker-certs:/certs/client \
  --volume jenkins-data:/var/jenkins_home \
  --publish 2376:2376 \
  docker:dind
```

# Build a Custom Jenkins Image with Docker and BlueOcean

```bash
FROM jenkins/jenkins:2.528.3-jdk21
USER root

# Install necessary dependencies and Docker CLI
RUN apt-get update && apt-get install -y lsb-release
RUN curl -fsSLo /usr/share/keyrings/docker-archive-keyring.asc \
  https://download.docker.com/linux/debian/gpg
RUN echo "deb [arch=$(dpkg --print-architecture) \
  signed-by=/usr/share/keyrings/docker-archive-keyring.asc] \
  https://download.docker.com/linux/debian \
  $(lsb_release -cs) stable" > /etc/apt/sources.list.d/docker.list
RUN apt-get update && apt-get install -y docker-ce-cli

# Install Jenkins plugins
USER jenkins
RUN jenkins-plugin-cli --plugins "blueocean docker-workflow json-path-api"


## build image
docker build -t myjenkins-blueocean:2.528.3-1 .
```

# Run Jenkins with Docker TLS Configuration

```bash
docker run --name jenkins-blueocean --restart=on-failure --detach \
  --network jenkins --env DOCKER_HOST=tcp://docker:2376 \
  --env DOCKER_CERT_PATH=/certs/client --env DOCKER_TLS_VERIFY=1 \
  --volume jenkins-data:/var/jenkins_home \
  --volume jenkins-docker-certs:/certs/client:ro \
  --publish 8082:8080 --publish 50000:50000 myjenkins-blueocean:2.528.3-1

```
# Add the TLS Certificates in Jenkins Credentials

```bash
docker cp jenkins-docker:/certs/client/ca.pem ./ca.pem
docker cp jenkins-docker:/certs/client/cert.pem ./cert.pem
docker cp jenkins-docker:/certs/client/key.pem ./key.pem

# Open Jenkins and go to Manage Jenkins > Manage Credentials.

# Create a new Global X.509 Client Certificate credential.

# Upload the ca.pem, cert.pem, and key.pem files that were copied from the jenkins-docker container.
```

# Jenkins Pipeline

```groovy
pipeline {
    agent any

    triggers {
        pollSCM('*/5 * * * *')
    }

    stages {
        stage('Dev') {
            steps {
                echo "Building..."

                withMaven(
                            maven: 'maven-3',
                            // Use `$WORKSPACE/.repository` for local repository folder to avoid shared repositories
                            mavenLocalRepo: '.repository'
                        ) {
                          sh "mvn clean verify"
                        }
            }
        }

        stage('Test') {
            steps {
                echo "Testing..."
                sh 'echo "Test Test stage"'
            }
        }

        stage('Code Deliver') {
            steps {
                sh 'echo "Test deliver stage"'
                // sh 'mvn install -Dmaven.test.skip=true'
            }
        }
    }
}

```