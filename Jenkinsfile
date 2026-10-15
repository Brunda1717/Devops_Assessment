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
                echo 'Checking required files...'

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
                    docker build \
                        -t ${IMAGE_NAME}:${IMAGE_TAG} .
                '''
            }
        }

        stage('5. Verify Docker Image') {
            steps {
                echo 'Verifying the Docker image...'

                sh '''
                    docker image inspect ${IMAGE_NAME}:${IMAGE_TAG}
                '''
            }
        }

        stage('6. Test Website Content') {
            steps {
                echo 'Testing MindBloom website content...'

                sh '''
                    grep -qi "MindBloom" index.html
                    grep -qi "meditation" index.html

                    echo "Website content checks passed."
                '''
            }
        }

        stage('7. Stop Previous Container') {
            steps {
                echo 'Removing the previous MindBloom container if it exists...'

                sh '''
                    if docker container inspect ${CONTAINER_NAME} >/dev/null 2>&1; then
                        docker stop ${CONTAINER_NAME} || true
                        docker rm ${CONTAINER_NAME} || true
                    fi
                '''
            }
        }

        stage('8. Deploy MindBloom') {
            steps {
                echo 'Deploying MindBloom using Docker...'

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
                echo 'Checking container status...'

                sh '''
                    docker ps --filter "name=^/${CONTAINER_NAME}$"

                    docker container inspect \
                        --format='{{.State.Running}}' \
                        ${CONTAINER_NAME} | grep -q true
                '''
            }
        }

        stage('10. Test Live Website') {
            steps {
                echo 'Testing the deployed website...'

                sh '''
                    for i in $(seq 1 15); do
                        if curl -fsS http://localhost:${HOST_PORT}/ -o /tmp/mindbloom-test.html; then
                            break
                        fi

                        sleep 2
                    done

                    test -s /tmp/mindbloom-test.html
                    grep -qi "MindBloom" /tmp/mindbloom-test.html

                    echo "Live website test passed."
                '''
            }
        }
    }

    post {
        success {
            echo '''
            ==========================================
                MINDBLOOM DEPLOYMENT SUCCESSFUL!
            ==========================================
            Website: http://localhost:8082
            Docker Image: mindbloom:1.0
            Container: mindbloom
            ==========================================
            '''
        }

        failure {
            echo 'MindBloom pipeline failed. Check the Jenkins console output.'
        }

        always {
            echo 'MindBloom CI/CD pipeline execution completed.'
        }
    }
}