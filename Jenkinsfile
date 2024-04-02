pipeline {
    agent none

    tools {
        // Install the Maven version configured as "M3" and add it to the path.
        maven "mymaven"
    }
    
    environment{
        BUILD_SERVER_IP='ec2-user@54.209.175.61'
        IMAGE_NAME='abithamanoaharan/my_image'
        DEPLOY_SERVER_IP='ec2-user@44.204.145.63'
    }

    stages {
        stage('Compile') {
            // agent {label "linux_slave"}
            agent any
            steps {              
              script{
                     echo "COMPILING"
                     sh "mvn compile"
              }             
            }
            
        }
        stage('Test') {
            agent any
            steps {           
              script{
                   echo "RUNNING THE TC"
                   sh "mvn test"
                }              
             
            }            
        
        post{
            always{
                junit 'target/surefire-reports/*.xml'
            }
        }
        }
        stage('Containerise the app') {
            agent any
            steps {              

                script{
                     echo "Creating the docker image and Push to registry"
                sshagent(['slave2']) {
                withCredentials([usernamePassword(credentialsId: 'docker_hub_credentials', passwordVariable: 'password', usernameVariable: 'username')]) {
                sh "scp -o StrictHostKeyChecking=no server-script.sh ${BUILD_SERVER_IP}:/home/ec2-user"
                sh "ssh -o StrictHostKeyChecking=no ${BUILD_SERVER_IP} 'bash server-script.sh ${IMAGE_NAME} ${BUILD_NUMBER}'"
                sh "ssh ${BUILD_SERVER_IP} sudo docker login -u ${username} -p ${password}"
                sh "ssh ${BUILD_SERVER_IP} sudo docker push ${IMAGE_NAME}:${BUILD_NUMBER}"
                }
                }             
                }
            }            
        }
         stage('Deploy the docker container on Test server') {
            agent any
            steps {            
                script{
                     echo "Deploy the container"
                sshagent(['slave2']) {
                withCredentials([usernamePassword(credentialsId: 'docker_hub_credentials', passwordVariable: 'password', usernameVariable: 'username')]) {
                sh "ssh -o StrictHostKeyChecking=no ${DEPLOY_SERVER_IP} sudo yum install docker -y"
                 sh "ssh -o StrictHostKeyChecking=no ${DEPLOY_SERVER_IP} sudo systemctl start docker"
                sh "ssh ${DEPLOY_SERVER_IP} sudo docker login -u ${username} -p ${password}"
                sh "ssh ${DEPLOY_SERVER_IP} sudo docker run -itd -P ${IMAGE_NAME}:${BUILD_NUMBER}"
                }
                }             
                }
            }            
        }

    }
}