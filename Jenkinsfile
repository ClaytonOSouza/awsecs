node {
    def app
    stage('Clone repository') {
        git branch: "master", url: "https://github.com/ClaytonOSouza/awsecs.git", credentialsId: "jenkins-example-github"
    }
    stage('Build image') {
        sh "docker build --build-arg APP_NAME=receipts -t 328527480917.dkr.ecr.us-east-1.amazonaws.com/sitemeu:latest -f docker/prod/Dockerfile ."
    }
    stage('Push image') {
        docker.withRegistry('https://328527480917.dkr.ecr.us-east-1.amazonaws.com', 'ecr:us-east-1:328527480917') {
            sh "docker push 328527480917.dkr.ecr.us-east-1.amazonaws.com/sitemeu:latest"
        }
    }
}
