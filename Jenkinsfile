pipeline{
    agent any
    environment{
        DEV_SERVER = "ec2-user@172.31.7.83"
        DEPLOY_SERVER = "ec2-user@172.31.6.244"
        IMAGE_NAME = "vadakanathurobin/myprivaterepo:php$BUILD_NUMBER"
    }
    stages{
        // build server/dev server --- stage 1
        stage("Build the docker image and push to docker hub"){
            steps{
                script{
                    //  Go to any Jenkins pipeline and use ssAgent to generate the below line
                    sshagent(['aws-linux-server-keypair']) {
                        // Go to any Jenkins pipeline and use With Credentials in pipeline syntax to generate the below line
                        withCredentials([usernamePassword(credentialsId: 'docker-hub', passwordVariable: 'DOCKERPASSWORD', usernameVariable: 'DOCKERUSERNAME')]) {
                            sh "scp -o strictHostKeyChecking=no -r devserverconfig ${DEV_SERVER}:/home/ec2-user"
                            sh "ssh -o strictHostKeyChecking=no ${DEV_SERVER} 'bash ~/devserverconfig/docker-script.sh'"
                            sh "ssh ${DEV_SERVER} sud docker build -t ${IMAGE_NAME} /home/ec2-user/devserverconfig"
                            sh "ssh ${DEV_SERVER} sudo docker login -u ${DOCKERUSERNAME} -p ${DOCKERPASSWORD}"
                            sh "ssh ${DEV_SERVER} sudo docker push ${IMAGE_NAME}"
                        }
                    }
                }
            }
        }

        stage("Run the php-db app with docker-compose"){
            // deploy with docker-compose on deploy server --- stage 2
            steps{
                script{
                    sshagent(['aws-linux-server-keypair']) {
                        // Go to any Jenkins pipeline and use With Credentials in pipeline syntax to generate the below line
                        withCredentials([usernamePassword(credentialsId: 'docker-hub', passwordVariable: 'DOCKERPASSWORD', usernameVariable: 'DOCKERUSERNAME')]) {
                            sh "scp -o strictHostKeyChecking=no -r testserverconfig ${DEPLOY_SERVER}:/home/ec2-user"
                            sh "ssh -o strictHostKeyChecking=no ${DEPLOY_SERVER} 'bash ~/testserverconfig/docker-script.sh'"
                            sh "ssh ${DEPLOY_SERVER} sudo docker login -u ${DOCKERUSERNAME} -p ${DOCKERPASSWORD}"
                            sh "ssh ${DEPLOY_SERVER} bash /home/ec2-user/testserverconfig/docker-compose-script.sh ${IMAGE_NAME}"
                        }
                }
            }
        }
        }

    }
}