pipeline {
  agent any

  environment {
    AWS_ACCOUNT_ID="168546287356"
    AWS_DEFAULT_REGION="us-east-1"
    CLUSTER_NAME="default"
    SERVICE_NAME="nodejs-container-service"
    TASK_DEFINITION_NAME="first-run-task-definition"
    DESIRED_COUNT="1"
    IMAGE_REPO_NAME="express-test"
    IMAGE_TAG="${env.BUILD_ID}"
    REPOSITORY_URI="${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}"
    registryCredential = "aws-admin-user"
  }

  stages {
    stage('Logging into AWS ECR') {
        steps {
          script {
             withCredentials([string(credentialsId: "${registryCredential}", variable: 'REGISTRY_CREDENTIALS')]) {
               sh "aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS -p ${REPOSITORY_URI}"
                  }
              }
          }
       }
    stage('Build') {
      steps {
        // Checkout the code from the Git repository
        checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: 'https://github.com/DDnova/express.git']]])


        // Build the Docker image
        sh "docker build -t ${REPOSITORY_URI}:${IMAGE_TAG} ."

        // Push the Docker image to ECR
        sh "docker push ${REPOSITORY_URI}:${IMAGE_TAG}"
      }
    }


    stage('Deploy to ECS') {
      steps {
        // Register a new task definition
        sh "aws ecs register-task-definition --family ${TASK_DEFINITION_NAME} --container-definitions '[{\"name\":\"${IMAGE_REPO_NAME}\",\"image\":\"${REPOSITORY_URI}:${IMAGE_TAG}\",\"portMappings\":[{\"containerPort\":3000}],\"essential\":true}]'"

        // Update the service on ECS to use the new task definition
        sh "aws ecs update-service --cluster ${CLUSTER_NAME} --service ${SERVICE_NAME} --force-new-deployment --desired-count ${DESIRED_COUNT} --task-definition ${TASK_DEFINITION_NAME}:${env.BUILD_ID}"
      }
    }
  }
}