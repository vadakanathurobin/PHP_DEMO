pipeline{
    agent any
    environment{
        DEV_SERVER = 'ec2-user@172.31.15.51'
        DEPLOY_SERVER = 'ec2-user@xxxxx'
        IMAGE_NAME='devopstrainer/java-mvn-private-repos:php$BUILD_NUMBER'
    }

    stages{
        // stage1 - build server/dev server 
        stage("Build the dockerimage and push to dockerhub"){
            steps{
                script{
                   sshagent(['aws-key']) {
                    withCredentials([usernamePassword(credentialsId: 'docker-hub', passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]) {
                    sh "scp -o strictHostKeyChecking=no -r devserverconfig ${DEV_SERVER}:/home/ec2-user" 
                    sh "ssh -o strictHostKeyChecking=no ${DEV_SERVER} 'bash ~/devserverconfig/docker-script.sh'"   
                    sh "ssh ${DEV_SERVER} sudo docker build -t ${IMAGE_NAME} /home/ec2-user/devserverconfig"
                    sh "ssh ${DEV_SERVER} sudo docker login -u ${USERNAME} -p ${PASSWORD}"
                    sh "ssh ${DEV_SERVER} sudo docker push ${IMAGE_NAME}"
                     
                   }
                }
            }
        }
        }
        stage("run the php-db app with dockercompose"){
            // deploy with compose on deploy server --stage 2 
              steps{
                script{
                     sshagent(['aws-key']) {
                        withCredentials([usernamePassword(credentialsId: 'docker-hub', passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]) {
                        sh "scp -o strictHostKeyChecking=no -r testserverconfig ${DEPLOY_SERVER}:/home/ec2-user"
                        sh "ssh -o strictHostKeyChecking=no ${DEPLOY_SERVER} 'bash ~/testserverconfig/docker-script.sh'
                        sh "ssh ${DEPLOY_SERVER} sudo docker login -u ${USERNAME} -p ${PASSWORD}"
                        sh "ssh ${DEPLOY_SERVER} bash /home/ec2-user/testserverconfig/docker-compose-script.sh ${IMAGE_NAME}"

                }
                }
                }
              }
        }
    }
}