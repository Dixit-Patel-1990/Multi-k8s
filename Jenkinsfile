pipeline {
    agent any

     environment {
        PATH = "/opt/homebrew/bin:$PATH"
        AWS_EB_CREDENTIALS_ID = 'AWS_INFO'
        AWS_EB_REGION = 'us-west-1'
        AWS_EB_APPLICATION_NAME = 'test1'
        AWS_EB_ENVIRONMENT_NAME = 'Test1-env'
        AWS_EB_S3_BUCKET = 'elasticbeanstalk-us-west-1-328079970834'
        AWS_EB_S3_KEY = 'multi-container'

        // VERSION="${BUILD_NUMBER}"
        // ZIP="docker-web-app.${BUILD_NUMBER}.zip"

        VERSION="v1"
        ZIP="multi-container-web-app.v1.zip"
    }

    stages {
        

        stage("Checkout") {
            steps {
                  git(
                  url: 'https://github.com/Dixit-Patel-1990/multi-container-docker.git',
                  branch: "main"
                )
            }
        }
        stage("Create Build for Client Application test cases and Tag Image") {
            steps {
                 sh "docker build -t dixitpatel1008/client-test:latest -f ./client/Dockerfile.dev ./client"
                //   sh "docker build -t dixitpatel1008/docker-web-app:${BUILD_NUMBER} -f Dockerfile.dev ."
            }
        }
        stage("Run test cases") {
            steps {
                 sh "docker run -e CI=true dixitpatel1008/client-test:latest npm run test"
            }
        }
        stage("Build images for client, api, worker, nginx") {
            steps {
                 sh "docker build -t dixitpatel1008/multi-container-client:latest ./client"
                 sh "docker build -t dixitpatel1008/multi-container-nginx:latest ./nginx"
                 sh "docker build -t dixitpatel1008/multi-container-server:latest ./server"
                 sh "docker build -t dixitpatel1008/multi-container-worker:latest ./worker"
            }
        }
        stage("Push Image to docker hub") {
            steps {
                 sh "docker push dixitpatel1008/multi-container-client:latest"
                 sh "docker push dixitpatel1008/multi-container-nginx:latest"
                 sh "docker push dixitpatel1008/multi-container-server:latest"
                 sh "docker push dixitpatel1008/multi-container-worker:latest"
            }
        }

        stage('Deploy to Elastic Beanstalk') {
            steps {
                script {
                    withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', accessKeyVariable: 'AWS_ACCESS_KEY_ID', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY', credentialsId: env.AWS_EB_CREDENTIALS_ID]]) {
                        sh '/opt/homebrew/bin/aws configure set aws_secret_access_key ${AWS_ACCESS_KEY_ID}'
                        sh '/opt/homebrew/bin/aws configure set aws_secret_access_key ${AWS_SECRET_ACCESS_KEY}'
                        sh '/opt/homebrew/bin/aws configure set region us-west-1'
                        sh 'zip -r $ZIP docker-compose.yml'
                        sh '/opt/homebrew/bin/aws s3 cp $ZIP s3://$AWS_EB_S3_BUCKET/$ZIP'
                        sh '/opt/homebrew/bin/aws elasticbeanstalk create-application-version --application-name $AWS_EB_APPLICATION_NAME --version-label $BUILD_NUMBER --source-bundle S3Bucket=$AWS_EB_S3_BUCKET,S3Key=$ZIP'
                        sh '/opt/homebrew/bin/aws elasticbeanstalk update-environment --environment-name $AWS_EB_ENVIRONMENT_NAME --version-label $BUILD_NUMBER'
                    }
                }
            }
        }
    }       
}
     
