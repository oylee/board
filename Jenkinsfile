pipeline {
    agent any
    environment {
            TIME_ZONE = 'Asia/Seoul'
            PROFILE = 'local'
            AWS_CREDENTIAL_NAME = 'aws-key'
            REGION="us-east-1"
            ECR_PATH = '074397164943.dkr.ecr.us-east-1.amazonaws.com'
            IMAGE_NAME = '074397164943.dkr.ecr.us-east-1.amazonaws.com/board'
            DEPLOY_Host = "44.204.22.142"
        }
    stages {
        stage('Pull Codes from Github'){
            steps{
                checkout scm
            }
        }
        stage('Build Codes by Gradle') {
            steps {
              sh "./gradlew clean build"
            }
        }
        stage('dockerizing project by dockerfile') {
             steps {
                sh '''
                   docker build -t $IMAGE_NAME:$BUILD_NUMBER .
                   docker tag $IMAGE_NAME:$BUILD_NUMBER $IMAGE_NAME:latest
                   '''
             }
             post {
                   success {
                        echo 'success dockerizing project'
                   }
                   failure {
                        error 'fail dockerizing project' // exit pipeline
                   }
             }
        }
        stage('upload aws ECR') {
                    steps {
                        script{

                            docker.withRegistry("https://$ECR_PATH", "ecr:$REGION:$AWS_CREDENTIAL_NAME") {
                              docker.image("$IMAGE_NAME:$BUILD_NUMBER").push()
                              docker.image("$IMAGE_NAME:latest").push()
                            }

                        }
                    }
                    post {
                        success {
                            echo 'success upload image'
                        }
                        failure {
                            error 'fail upload image' // exit pipeline
                        }
                    }
                }
        stage('Deploy to AWS EC2 VM'){
             steps{
                sshagent(credentials : ["deploy-ssh-key"]) {
                    sh "ssh -o StrictHostKeyChecking=no ubuntu@$DEPLOY_Host \
                     'aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin $ECR_PATH; \
                    docker run -d -p 80:8080 -t $IMAGE_NAME:${BUILD_NUMBER};'"
                }
             }
        }

    }
}
