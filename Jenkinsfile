node {
    def app
    stage('Clone repository') {
        git branch: "master", url: "https://github.com/ClaytonOSouza/awsecs.git", credentialsId: "jenkins-example-github"
    }
    stage('Build image') {
        sh "docker build --build-arg APP_NAME=sitemeu -t 328527480917.dkr.ecr.us-east-1.amazonaws.com/sitemeu:$BUILD_NUMBER -f  Dockerfile ."
    }
    stage('Push image') {
        docker.withRegistry('https://328527480917.dkr.ecr.us-east-1.amazonaws.com', 'ecr:us-east-1:328527480917') {
            sh "docker push 328527480917.dkr.ecr.us-east-1.amazonaws.com/sitemeu:$BUILD_NUMBER"
        }
    }
    stage('Deploy') {
        sh "- echo $REPOSITORY_URL:$IMAGE_TAG" {
        sh "TASK_DEFINITION=$aws ecs describe-task-definition --task-definition "$TASK_DEFINTION_NAME" --region "$REGION""
        sh "NEW_CONTAINER_DEFINTIION=$echo $TASK_DEFINITION | jq --arg IMAGE "$REPOSITORY_URL":"$IMAGE_TAG" '.taskDefinition.containerDefinitions[0].image = $IMAGE | .taskDefinition.containerDefinitions[0]'"
        sh "- echo "Registering new container definition...""
        sh "aws ecs register-task-definition --region "${REGION}" --family "${TASK_DEFINTION_NAME}" --container-definitions "${NEW_CONTAINER_DEFINTIION}""
        sh "echo "Updating the service...""
        sh "aws ecs update-service --region "${REGION}" --cluster "${CLUSTER_NAME}" --service "${SERVICE_NAME}"  --task-definition "${TASK_DEFINTION_NAME}""
        }       
    }
}

