pipeline {
    agent any

    tools {
        maven 'maven3'
    }

    environment {
        IMAGE_NAME = "techgryphdocker/boardgame-app"
    }

    stages {

        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('SonarQube Scan') {
            environment {
                SONAR_TOKEN = credentials('sonar-token')
            }
            steps {
                sh '''
                mvn sonar:sonar \
                -Dsonar.host.url=http://sonarqube:9000 \
                -Dsonar.login=$SONAR_TOKEN
                '''
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
    }
}

