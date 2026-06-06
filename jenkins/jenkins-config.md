Remote server deployment
```bash
pipeline {
   agent any

   options {
       timestamps()
       disableConcurrentBuilds()
       buildDiscarder(logRotator(numToKeepStr: '20'))
   }

   environment {
       // Source Code
       GIT_BRANCH = "jenkins-test/qa"
       GIT_URL = "git_project_url"
       GIT_CREDENTIALS = "jenkins_git_credential_id"

       // nexus or docker registry
       REGISTRY = "docker_repository_url"
       REGISTRY_CREDENTIALS = "jenkins_registry_id"

       // Application
       APP_NAME = "app-qa"
       IMAGE = "${REGISTRY}/${APP_NAME}"
       IMAGE_TAG = "${BUILD_NUMBER}"

       // QA Server
       QA_HOST = "server_ip"
       QA_DEPLOY_PATH = "/home/app/jenkinstest"
       QA_CREDENTIALS = "jenkins_server_cred_id"
   }

   stages {

       stage('Checkout') {
           steps {
               git(
                   branch: env.GIT_BRANCH,
                   credentialsId: env.GIT_CREDENTIALS,
                   url: env.GIT_URL
               )
           }
       }

       stage('Build & Test') {
           steps {
               sh '''
                   mvn clean compile
                   mvn test
                   mvn package -DskipTests
               '''
           }
       }

       stage('Docker Build') {
           steps {
               sh '''
                   docker build \
                       -t ${IMAGE}:${IMAGE_TAG} .
               '''
           }
       }

       stage('Push To Registry') {
           steps {
               withCredentials([
                   usernamePassword(
                       credentialsId: env.REGISTRY_CREDENTIALS,
                       usernameVariable: 'REG_USER',
                       passwordVariable: 'REG_PASS'
                   )
               ]) {
                   sh '''
                       echo "$REG_PASS" | docker login ${REGISTRY} \
                           -u "$REG_USER" \
                           --password-stdin

                       docker push ${IMAGE}:${IMAGE_TAG}

                       docker logout ${REGISTRY}
                   '''
               }
           }
       }

       stage('Deploy QA') {
           steps {

               withCredentials([
                   usernamePassword(
                       credentialsId: env.REGISTRY_CREDENTIALS,
                       usernameVariable: 'REG_USER',
                       passwordVariable: 'REG_PASS'
                   ),
                   usernamePassword(
                       credentialsId: env.QA_CREDENTIALS,
                       usernameVariable: 'DEPLOY_USER',
                       passwordVariable: 'DEPLOY_PASS'
                   )
               ]) {

              sh '''
                       sshpass -p "$DEPLOY_PASS" ssh -o StrictHostKeyChecking=no "$DEPLOY_USER@$QA_HOST" \
                         "export IMAGE='${IMAGE}' TAG='${IMAGE_TAG}' REGISTRY='${REGISTRY}' REG_USER='$REG_USER' REG_PASS='$REG_PASS' QA_DEPLOY_PATH='${QA_DEPLOY_PATH}'; \
                          echo \\"\\$REG_PASS\\" | docker login \\"\\$REGISTRY\\" -u \\"\\$REG_USER\\" --password-stdin && \
                          cd \\"\\$QA_DEPLOY_PATH\\" && \
                          docker compose pull && \
                          docker compose up -d && \
                          docker logout \\"\\$REGISTRY\\" && \
                          docker images \\"\\$IMAGE\\" --format '{{.Repository}}:{{.Tag}}' | grep -v \\":\\$TAG\\$\\" | xargs -r docker rmi -f || true"
                   '''
               }
           }
       }
   }

   post {

       success {
       	echo "BUILD SUCCESS"
       }

       failure {
       	echo "BUILD FAILED"
       }

       always {
           cleanWs()
       }
   }
}

```