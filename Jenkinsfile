@library('lucien-shared-library')
pipeline {
    environment {
        IMAGE_NAME = 'webapp'
        IMAGE_TAG = 'v1'
        DOCKER_PASSWORD = credentials('docker-password')
        DOCKER_USERNAME = 'lucien'
        HOST_PORT = 80
        CONTAINER_PORT = 80
        IP_DOCKER = '172.17.0.1'
    }
    
    agent any 
    
    stages {
        stage('Build') {
            steps {
                script {
                    sh '''
                        docker build --no-cache -t $IMAGE_NAME:$IMAGE_TAG .
                    '''
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    sh '''
                        docker run --rm -dp $HOST_PORT:$CONTAINER_PORT --name $IMAGE_NAME $IMAGE_NAME:$IMAGE_TAG
                        sleep 5
                        curl -I http://$IP_DOCKER
                        sleep 5
                        docker stop $IMAGE_NAME
                    '''
                }
            }
        }

        stage('Release') {
            steps {
                script {
                    sh '''
                        docker tag $IMAGE_NAME:$IMAGE_TAG $DOCKER_USERNAME/$IMAGE_NAME:$IMAGE_TAG
                        echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin
                        docker push $DOCKER_USERNAME/$IMAGE_NAME:$IMAGE_TAG
                    '''
                }
            }
        }

        stage('Deploy Review') {
            environment {
                SERVER_IP = '50.19.38.11'
                SERVER_USERNAME = 'ubuntu'
            }
            steps {
                script {
                    timeout(time: 30, unit: "MINUTES") {
                        input message: "Voulez-vous réaliser un déploiement sur l'environnement de review ?", ok: 'Yes'
                    }
                    sshagent(['key-pair']) {
                        sh '''
                            echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin
                            ssh -o StrictHostKeyChecking=no -l $SERVER_USERNAME $SERVER_IP "docker rm -f $IMAGE_NAME || echo 'All deleted'"
                            ssh -o StrictHostKeyChecking=no -l $SERVER_USERNAME $SERVER_IP "docker pull $DOCKER_USERNAME/$IMAGE_NAME:$IMAGE_TAG || echo 'Image Download successfully'"
                            sleep 30
                            ssh -o StrictHostKeyChecking=no -l $SERVER_USERNAME $SERVER_IP "docker run --rm -dp $HOST_PORT:$CONTAINER_PORT --name $IMAGE_NAME $DOCKER_USERNAME/$IMAGE_NAME:$IMAGE_TAG"
                            sleep 5
                            curl -I http://$SERVER_IP:$HOST_PORT
                        '''
                    }
                }
            }
        }

        stage('Deploy Staging') {
            environment {
                SERVER_IP = '3.90.114.53'
                SERVER_USERNAME = 'ubuntu'
            }
            steps {
                script {
                    timeout(time: 30, unit: "MINUTES") {
                        input message: "Voulez-vous réaliser un déploiement sur l'environnement de staging ?", ok: 'Yes'
                    }
                    sshagent(['key-pair']) {
                        sh '''
                            echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin
                            ssh -o StrictHostKeyChecking=no -l $SERVER_USERNAME $SERVER_IP "docker rm -f $IMAGE_NAME || echo 'All deleted'"
                            ssh -o StrictHostKeyChecking=no -l $SERVER_USERNAME $SERVER_IP "docker pull $DOCKER_USERNAME/$IMAGE_NAME:$IMAGE_TAG || echo 'Image Download successfully'"
                            sleep 30
                            ssh -o StrictHostKeyChecking=no -l $SERVER_USERNAME $SERVER_IP "docker run --rm -dp $HOST_PORT:$CONTAINER_PORT --name $IMAGE_NAME $DOCKER_USERNAME/$IMAGE_NAME:$IMAGE_TAG"
                            sleep 5
                            curl -I http://$SERVER_IP:$HOST_PORT
                        '''
                    }
                }
            }
        }

        stage('Deploy Prod') {
            // when {expression {GIT_BRANCH == 'origin/prod'}}
            environment {
                SERVER_IP = '100.27.214.118'
                SERVER_USERNAME = 'ubuntu'
            }
            steps {
                script {
                    timeout(time: 30, unit: "MINUTES") {
                        input message: "Voulez-vous réaliser un déploiement sur l'environnement de production ?", ok: 'Yes'
                    }
                    sshagent(['key-pair']) {
                        sh '''
                            echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin
                            ssh -o StrictHostKeyChecking=no -l $SERVER_USERNAME $SERVER_IP "docker rm -f $IMAGE_NAME || echo 'All deleted'"
                            ssh -o StrictHostKeyChecking=no -l $SERVER_USERNAME $SERVER_IP "docker pull $DOCKER_USERNAME/$IMAGE_NAME:$IMAGE_TAG || echo 'Image Download successfully'"
                            sleep 30
                            ssh -o StrictHostKeyChecking=no -l $SERVER_USERNAME $SERVER_IP "docker run --rm -dp $HOST_PORT:$CONTAINER_PORT --name $IMAGE_NAME $DOCKER_USERNAME/$IMAGE_NAME:$IMAGE_TAG"
                            sleep 5
                            curl -I http://$SERVER_IP:$HOST_PORT
                        '''
                    }
                }
            }
        }
    }

    post {
        always {
            script {
                slackNotifier currentBuild.result
            }
        }
    }
}