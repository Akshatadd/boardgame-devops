pipeline {
    agent any

    tools {
        maven 'maven3'
    }

    environment {
        IMAGE_NAME = "akshatadd/boardgame-app"
    }

    stages {

        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('SonarQube Analysis') {
    steps {
        withSonarQubeEnv('sonar') {
            sh '''
              sonar-scanner \
              -Dsonar.projectKey=boardgame-app \
              -Dsonar.sources=src \
              -Dsonar.java.binaries=target
            '''
        }
    }
}

        stage('Docker Build') {
            steps {
                sh 'docker build -t $IMAGE_NAME:$BUILD_NUMBER .'
            }
        }

        stage('Docker Push') {
            environment {
                DOCKER_CREDS = credentials('dockerhub-creds')
            }
            steps {
                sh '''
                echo $DOCKER_CREDS_PSW | docker login -u $DOCKER_CREDS_USR --password-stdin
                docker push $IMAGE_NAME:$BUILD_NUMBER
                '''
            }
        }

        stage('Deploy to App Servers') {
            steps {
                withCredentials([sshUserPrivateKey(
                    credentialsId: 'ec2-ssh-key',
                    keyFileVariable: 'SSH_KEY',
                    usernameVariable: 'SSH_USER'
                )]) {
                    sh '''
                    ssh -i $SSH_KEY -o StrictHostKeyChecking=no $SSH_USER@3.109.60.11 "
                      docker pull $IMAGE_NAME:$BUILD_NUMBER &&
                      docker rm -f boardgame || true &&
                      docker run -d --name boardgame -p 8080:8080 $IMAGE_NAME:$BUILD_NUMBER
                    "

                    ssh -i $SSH_KEY -o StrictHostKeyChecking=no $SSH_USER@13.201.38.247 "
                      docker pull $IMAGE_NAME:$BUILD_NUMBER &&
                      docker rm -f boardgame || true &&
                      docker run -d --name boardgame -p 8080:8080 $IMAGE_NAME:$BUILD_NUMBER
                    "
                    '''
                }
            }
        }

    }
}
