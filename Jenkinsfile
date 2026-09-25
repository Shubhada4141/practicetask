
pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'shubhadashingane/new-practice-task'
        CONTAINER_NAME = 'new-task-con'
        APP_PORT = '8081'
    }

    stages {

        stage('Build') {
            steps {
                echo '===== BUILDING APPLICATION ====='
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Test') {
            steps {
                echo '===== RUNNING TESTS ====='
                sh 'mvn test'
            }
        }

        stage('Docker Build') {
            steps {
                echo '===== BUILDING DOCKER IMAGE ====='
                sh 'docker build -t $DOCKER_IMAGE .'
            }
        }

        stage('Docker Login & Push') {
            steps {
                echo '===== PUSHING IMAGE TO DOCKER HUB ====='

                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {
                    sh '''
                        echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin
                        docker push $DOCKER_IMAGE
                        docker logout
                    '''
                }
            }
        }

        stage('Deploy') {
            steps {
                echo '===== DEPLOYING APPLICATION ====='

                sh '''
                    docker pull $DOCKER_IMAGE

                    docker stop $CONTAINER_NAME || true
                    docker rm $CONTAINER_NAME || true

                    docker run -d \
                        --name $CONTAINER_NAME \
                        -p $APP_PORT:$APP_PORT \
                        --restart unless-stopped \
                        $DOCKER_IMAGE

                    docker ps
                '''
            }
        }
    }

    post {
        success {
            echo '======================================'
            echo 'PIPELINE COMPLETED SUCCESSFULLY'
            echo 'APPLICATION DEPLOYED'
            echo '======================================'
        }

        failure {
            echo '======================================'
            echo 'PIPELINE FAILED'
            echo 'CHECK THE STAGE LOG ABOVE'
            echo '======================================'
        }
    }
}

