pipeline {
    agent {label 'ec2_private_slave'}
    environment {
        env = 'dev'
        rds_port = 3306
        }
    stages {
        stage('CI') {
            steps {
                // Build the Dockerfile
                sh "docker build . -t zainabsabry/jenkins_node_rds_redis:latest"
                // Push the image to our account

                //first login to docker
                withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]){

                    sh "docker login -u ${USERNAME} -p ${PASSWORD}"

                }

                //now, push the image
                sh "docker push zainabsabry/jenkins_node_rds_redis:latest"
                
            }
        post {
            // If CI stage was successful, send a green notification to slack
            success {
                slackSend(channel: 'jenkins', color: 'good', message: "CI was successful - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
            }
            // If deployment fails, send a red notification
            failure {
                slackSend(channel: 'jenkins', color: 'danger', message: "CI Failed  - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
            }
        }
        }    
        stage('CD') {
            environment {
                RDS_ENDPOINT = '$(aws ssm get-parameter --name /dev/database/endpoint --query "Parameter.Value" --with-decryption --output text)'
                RDS_USERNAME = '$(aws ssm get-parameter --name /dev/database/username --query "Parameter.Value" --with-decryption --output text)'
                RDS_PASSWORD = '$(aws ssm get-parameter --name /dev/database/password --query "Parameter.Value" --with-decryption --output text)'
                REDIS_ENDPOINT = '$(aws ssm get-parameter --name /dev/redis/endpoint --query "Parameter.Value" --with-decryption --output text)'
            }
            steps {
            withAWS(credentials: 'aws', region: 'us-east-1'){
                //Run the docker image with 
                sh "docker run -d -it -p 3000:3000 --env RDS_HOSTNAME=${RDS_ENDPOINT} --env RDS_USERNAME=${RDS_USERNAME} --env RDS_PASSWORD=${RDS_PASSWORD} --env RDS_PORT=${rds_port} --env REDIS_HOSTNAME=${REDIS_ENDPOINT}  zainabsabry/jenkins_node_rds_redis:latest"
            }
                
            }
        }
    }
    post {
        // If deployment was successful, send a green notification to slack
        success {
            slackSend(channel: 'jenkins', color: 'good', message: "Deployed Successfully - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
        }
        // If deployment fails, send a red notification
        failure {
            slackSend(channel: 'jenkins', color: 'danger', message: "Build Failed  - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
        }
        // trigger every-works
        always {
            slackSend message: 'New Build'
        }
    }
}
