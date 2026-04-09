pipeline{
  agent any

  environment {
    DOCKER_CREDS = credentials('docker-hub-credentials')
    IMAGE_TAG = "${BUILD_NUMBER}"
    DOCKER_IMAGE = "${DOCKER_CREDS_USR}/static-website"

    // EC2 details
    EC2_PROD_IP = "52.200.9.155"
    EC2_USER = "ubuntu"
    EC2_KEY_PROD = credentials('ec2-prod-key')

    CONTAINER_PORT = "80"
    HOST_PORT = "80"
    CONTAINER_NAME = "my-website"
  }
stages{
    stage ('cloning repo'){
      steps {
        sh '''
      rm -r static_application
      git clone https://github.com/khraj/static_application.git
      cd static_application
      git checkout main
      ls
      '''
    }

    }
    stage ('Docker image build'){
      steps {
        sh '''
        cd static-website
        docker build -t ${DOCKER_IMAGE}:${IMAGE_TAG} .
        docker tag ${DOCKER_IMAGE}:${IMAGE_TAG} ${DOCKER_IMAGE}:latest
        echo "Docker Image was built successully"
        docker images | grep static-website
        '''
      }
    }  
      stage('Push to dockerhub'){
      steps {
        sh '''
        echo ${DOCKER_CREDS_PSW} | docker login -u ${DOCKER_CREDS_USR} --password-stdin
        docker push ${DOCKER_IMAGE}:${IMAGE_TAG}
        docker push ${DOCKER_IMAGE}:latest
        docker logout
        '''
      }
    }

  stage('Depoy to production environment'){
      steps {
                script {
                    try {
                        sh '''
                            ssh -o StrictHostKeyChecking=no \
                                -i ${EC2_KEY_PRPD} \
                                ${EC2_USER}@${EC2_PROD_IP}
                                
                                echo "Pulling latest image..."
                                docker pull ${DOCKER_IMAGE}:latest
                                
                                echo "Stopping old container..."
                                docker stop ${CONTAINER_NAME} 2>/dev/null || true
                                docker rm ${CONTAINER_NAME} 2>/dev/null || true
                                
                                echo "Starting new container..."
                                docker run -d \
                                    -p ${HOST_PORT}:${CONTAINER_PORT} \
                                    --restart always \
                                    --name ${CONTAINER_NAME} \
                                    -e BUILD_NUMBER=${BUILD_NUMBER} \
                                    ${DOCKER_IMAGE}:latest
                                
                                echo "✓ Staging deployment completed"
                                docker ps | grep ${CONTAINER_NAME}
                            '
                        '''
                    } catch (Exception e) {
                        error("Staging deployment failed: ${e.message}")
                    }
                }
      }
    }    
}
}
