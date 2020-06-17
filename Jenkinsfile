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
    /*
      / These steps to create new revision of the TaskDefinition, then -
      / update the servie with the new TaskDefinition revision to deploy the image
      */
   stage("Deploy") {
        // Replace BUILD_TAG placeholder in the task-definition file -
        // with the remoteImageTag (imageTag-BUILD_NUMBER)
        // “s;%BUILD_NUMBER%;${BUILD_NUMBER};g”
        sh  "                                                                     \
          echo $BUILD_TAG"                             \
                                
        
        // Get current [TaskDefinition#revision-number]
        def currTaskDef = sh (
          returnStdout: true,
          script:  "                                                              \
            aws ecs describe-task-definition  --task-definition ${taskFamily}     \
                                              | egrep 'revision'                  \
                                              | tr ',' ' '                        \
                                              | awk '{print \$2}'                 \
          "
        ).trim()

        def currentTask = sh (
          returnStdout: true,
          script:  "                                                              \
            aws ecs list-tasks  --cluster ${clusterName}                          \
                                --family ${taskFamily}                            \
                                --output text                                     \
                                | egrep 'TASKARNS'                                \
                                | awk '{print \$2}'                               \
          "
        ).trim()

        /*
        / Scale down the service
        /   Note: specifiying desired-count of a task-definition in a service -
        /   should be fine for scaling down the service, and starting a new task,
        /   but due to the limited resources (Only one VM instance) is running
        /   there will be a problem where one container is already running/VM,
        /   and using a port(80/443), then when trying to update the service -
        /   with a new task, it will complaine as the port is already being used,
        /   as long as scaling down the service/starting new task run simulatenously
        /   and it is very likely that starting task will run before the scaling down service finish
        /   so.. we need to manually stop the task via aws ecs stop-task.
        */
        if(currTaskDef) {
          sh  "                                                                   \
            aws ecs update-service  --cluster ${clusterName}                      \
                                    --service ${serviceName}                      \
                                    --task-definition ${taskFamily}:${currTaskDef}\
                                    --desired-count 0                             \
          "
        }
        if (currentTask) {
          sh "aws ecs stop-task --cluster ${clusterName} --task ${currentTask}"
        }

        // Register the new [TaskDefinition]
        sh  "                                                                     \
          aws ecs register-task-definition  --family ${taskFamily}                \
                                            --cli-input-json ${taskDefile}        \
        "

        // Get the last registered [TaskDefinition#revision]
        def taskRevision = sh (
          returnStdout: true,
          script:  "                                                              \
            aws ecs describe-task-definition  --task-definition ${taskFamily}     \
                                              | egrep 'revision'                  \
                                              | tr ',' ' '                        \
                                              | awk '{print \$2}'                 \
          "
        ).trim()

        // ECS update service to use the newly registered [TaskDefinition#revision]
        //
        sh  "                                                                     \
          aws ecs update-service  --cluster ${clusterName}                        \
                                  --service ${serviceName}                        \
                                  --task-definition ${taskFamily}:${taskRevision} \
                                  --desired-count 1                               \
        "
      }
      
    stage("BUILD SUCCEED") {
        slackSend (color: '#00FF00', message: "SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
      }
   }    
      
