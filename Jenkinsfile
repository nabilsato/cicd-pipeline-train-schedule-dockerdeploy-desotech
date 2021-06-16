pipeline {
    agent any
    environment {
        DOCKER_IMAGE_NAME = 'nabil/desotech-schedule'
        DOCKER_IMAGE_TAG = 'latest'
        DOCKER_REGISTRY = 'https://r.deso.tech'
        DOCKER_NODE_IP = '10.10.139.22'
        TARGET_PORT = 8080
        EXTERNAL_PORT = 9000
    }
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('Build Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    app = docker.build("${DOCKER_IMAGE_NAME}")
                    app.inside {
                        sh 'echo $(curl localhost:${TARGET_PORT})'
                    }
                }
            }
        }
        stage('Push Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    docker.withRegistry("${DOCKER_REGISTRY}", 'docker_hub_login') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("${DOCKER_IMAGE_TAG}")
                    }
                }
            }
        }
        stage('DeployToProduction') {
            when {
                branch 'master'
            }
            steps {
                input 'Deploy to Production?'
                milestone(1)
                withCredentials([usernamePassword(credentialsId: 'webserver_login', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {
                    script {
                        sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$DOCKER_NODE_IP \"docker pull ${DOCKER_REGISTRY}/${DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER}\""
                        try {
                            sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$DOCKER_NODE_IP \"docker stop train-schedule\""
                            sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$DOCKER_NODE_IP \"docker rm train-schedule\""
                        } catch (err) {
                            echo: 'caught error: $err'
                        }
                        sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$DOCKER_NODE_IP \"docker run --restart always --name train-schedule -p $EXTERNAL_PORT:$TARGET_PORT -d ${DOCKER_REGISTRY}/${DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER}\""
                    }
                }
            }
        }
    }
}
