pipeline {
    agent any

    environment {
        IMAGE_NAME = 'mindbloom'
        IMAGE_TAG = '1.0'
        CONTAINER_NAME = 'mindbloom'
        HOST_PORT = '8082'
        CONTAINER_PORT = '80'
    }

    stages {

        stage('1. Checkout Source Code') {
            steps {
                echo 'Checking out MindBloom source code...'
                checkout scm
            }
        }

        stage('2. Check Docker') {
            steps {
                echo 'Checking Docker installation...'
                sh 'docker --version'
            }
        }

        stage('3. Validate Project Files') {
            steps {
                echo 'Checking required project files...'

                sh '''
                    test -f Dockerfile
                    test -f index.html
                    test -f Jenkinsfile

                    echo "All required files are present."
                '''
            }
        }

        stage('4. Build Docker Image') {
            steps {
                echo 'Building MindBloom Docker image...'

                sh '''
                    docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                '''
            }
        }

        stage('5. Verify Docker Image') {
            steps {
                echo 'Verifying Docker image...'

                sh '''
                    docker image inspect ${IMAGE_NAME}:${IMAGE_TAG} > /dev/null

                    echo "Docker image verified successfully."
                '''
            }
        }

        stage('6. Test Website Content') {
            steps {
                echo 'Testing MindBloom HTML content...'

                sh '''
                    grep -qi "MindBloom" index.html
                    grep -qi "meditation" index.html

                    echo "Website content tests passed."
                '''
            }
        }

        stage('7. Stop Previous Container') {
            steps {
                echo 'Checking for an existing MindBloom container...'

                sh '''
                    if docker container inspect ${CONTAINER_NAME} >/dev/null 2>&1; then
                        docker stop ${CONTAINER_NAME} || true
                        docker rm ${CONTAINER_NAME} || true
                    fi

                    echo "Previous container cleanup completed."
                '''
            }
        }

        stage('8. Deploy MindBloom') {
            steps {
                echo 'Deploying MindBloom...'

                sh '''
                    docker run -d \
                        --name ${CONTAINER_NAME} \
                        --restart unless-stopped \
                        -p ${HOST_PORT}:${CONTAINER_PORT} \
                        ${IMAGE_NAME}:${IMAGE_TAG}
                '''
            }
        }

        stage('9. Verify Deployment') {
            steps {
                echo 'Verifying MindBloom container status...'

                sh '''
                    docker container inspect \
                        --format='{{.State.Running}}' \
                        ${CONTAINER_NAME} | grep -q true

                    echo "MindBloom container is running."
                '''
            }
        }

        stage('10. Test Live Website') {
            steps {
                echo 'Testing the website inside the MindBloom container...'

                sh '''
                    SUCCESS=0

                    for i in $(seq 1 15); do
                        if docker exec ${CONTAINER_NAME} \
                            wget -q -O /tmp/mindbloom-test.html http://localhost/; then
                            SUCCESS=1
                            break
                        fi

                        sleep 2
                    done

                    if [ "$SUCCESS" -ne 1 ]; then
                        echo "ERROR: MindBloom website did not respond."
                        docker logs ${CONTAINER_NAME}
                        exit 1
                    fi

                    docker exec ${CONTAINER_NAME} \
                        grep -qi "MindBloom" /tmp/mindbloom-test.html

                    echo "MindBloom website HTTP test passed."
                '''
            }
        }
    }

    post {
        success {
            echo '''
            ==========================================
              MINDBLOOM CI/CD PIPELINE SUCCESSFUL!
            ==========================================
              Docker Image: mindbloom:1.0
              Container: mindbloom
              Website: http://localhost:8082
            ==========================================
            '''
        }

        failure {
            echo 'MindBloom pipeline failed. Check Console Output.'
        }

        always {
            echo 'MindBloom CI/CD pipeline execution completed.'
        }
    }
}