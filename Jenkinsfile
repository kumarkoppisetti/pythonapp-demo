pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = 'dockerhub-creds'  // ID of Jenkins credential
        DOCKER_IMAGE = 'kumarkoppisetti/flask-app' 
        // SONARQUBE_ENV = 'SonarQube'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/kumarkoppisetti/pythonapp-demo.git' 
            }
        }

        stage('SonarQube Scan') {
            steps {
                withCredentials([string(credentialsId: 'sonarqube-token', variable: 'SONAR_TOKEN')]) {
                    sh '''
                           /opt/sonar-scanner/bin/sonar-scanner \
                          -Dsonar.projectName=python-app-demo \
                          -Dsonar.projectKey=python-flask-app \
                          -Dsonar.sources=. \
                          -Dsonar.host.url=http://localhost:9000/ \
                          -Dsonar.token=sqa_87751d8aaf386c53e837aeca41111f9979f776b0
                    '''
                }
            }
        }

        
        stage('Build Docker Image') {
            steps {
                script {                    
                    sh "docker build -t $DOCKER_IMAGE:${BUILD_NUMBER} ."                    
                }
            }
        }

        stage('Scan with Trivy') {
            steps {
                script {
                    sh """
                        trivy image --exit-code 0 --severity HIGH,CRITICAL --format table $DOCKER_IMAGE:$BUILD_NUMBER || true
                    """
                }
            }
        }

        stage('DockerHub Login') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                    echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                    """
                }
            }
        }       

        stage('Push Image') {
            steps {               
                sh "docker push $DOCKER_IMAGE:${BUILD_NUMBER}"
                
            }
        }
    
        stage('Deploy to EC2') {
            steps {
                sshagent(['EC2_SSH_KEY']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ec2-user@34.203.215.49 '
                            docker pull kumarkoppisetti/flask-app:$BUILD_NUMBER &&
                            docker stop flask-app || true &&
                            docker rm flask-app || true &&
                            docker run -d -p 5000:5000 --name flask-app kumarkoppisetti/flask-app:$BUILD_NUMBER
                        '
                    """
                }
            }
        }
    }
    
}
