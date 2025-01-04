pipeline {
   agent none
  environment{
       BUILD_SERVER_IP='ec2-user@172.31.2.225'
       IMAGE_NAME='jackdhub/jdk-mvn-addressbook:php$BUILD_NUMBER'
       DEPLOY_SERVER_IP='ec2-user@172.31.15.47'
   }
    stages {          
        stage('BUILD DOCKERIMAGE AND PUSH TO DOCKERHUB') {
            agent any            
            steps {
                script{
                sshagent(['slave2']) {
                withCredentials([usernamePassword(credentialsId: 'docker-hub', passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]) {
                echo "Packaging the apps"
                sh "scp -o StrictHostKeyChecking=no -r devconfig ${BUILD_SERVER_IP}:/home/ec2-user"
                sh "ssh -o StrictHostKeyChecking=no ${BUILD_SERVER_IP} 'bash ~/devconfig/docker-script.sh'"
                sh "ssh ${BUILD_SERVER_IP} sudo docker build -t ${IMAGE_NAME} /home/ec2-user/devconfig/"
                //sh "ssh ${BUILD_SERVER_IP} 'echo ${PASSWORD} | sudo docker login --username ${USERNAME} --password-stdin'"
                sh "ssh ${BUILD_SERVER_IP} sudo docker login -u $USERNAME -p $PASSWORD"
                sh "ssh ${BUILD_SERVER_IP} sudo docker push ${IMAGE_NAME}"
              }
            }
            }
        }
        }
        stage('Deploy Docker Container Using Docker Compose') {
            agent any
            steps {
                script {
                    sshagent(['slave3']) {
                        withCredentials([usernamePassword(credentialsId: 'docker-hub', passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]) {
                            sh "scp -o StrictHostKeyChecking=no -r deployconfig ${DEPLOY_SERVER_IP}:/home/ec2-user"
                            sh "ssh -o StrictHostKeyChecking=no ${DEPLOY_SERVER_IP} 'bash ~/deployconfig/docker-script.sh'"
                            sh """
                                ssh -o StrictHostKeyChecking=no ${DEPLOY_SERVER_IP} << EOF
                                echo ${PASSWORD} | sudo docker login --username ${USERNAME} --password-stdin
                                bash /home/ec2-user/deployconfig/docker-compose-script.sh ${IMAGE_NAME}
                                EOF
                            """
                        }
                    }
               }
           }
       }
    }
}