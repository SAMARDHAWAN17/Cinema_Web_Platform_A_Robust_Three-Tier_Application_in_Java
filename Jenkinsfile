pipeline {
    agent { label 'slave' }
    
    tools {
        maven 'MAVEN'
        jdk 'JAVA'
    }
    
    environment {
        SONAR_HOME = tool 'sonar-scanner'
        DOCKER_IMAGE = 'samardhawan17/cinema-app'
    }
    
    stages {
        
        stage('Checkout') {
            steps {
                git branch: 'main', 
                    url: 'https://github.com/SAMARDHAWAN17/Cinema_Web_Platform_A_Robust_Three-Tier_Application_in_Java.git'
            }
        }
        
        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh '''
                        $SONAR_HOME/bin/sonar-scanner \
                        -Dsonar.projectKey=cinema-app \
                        -Dsonar.projectName=cinema-app \
                        -Dsonar.java.binaries=target/classes
                    '''
                }
            }
        }
        
        stage('Docker Build') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE:$BUILD_NUMBER .'
                sh 'docker tag $DOCKER_IMAGE:$BUILD_NUMBER $DOCKER_IMAGE:latest'
            }
        }

        stage('Trivy Scan') {
            steps {
                sh '''
                trivy image --no-progress --severity HIGH,CRITICAL \
                $DOCKER_IMAGE:$BUILD_NUMBER
                '''
            }
        }
        
        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                    sh 'docker push $DOCKER_IMAGE:$BUILD_NUMBER'
                    sh 'docker push $DOCKER_IMAGE:latest'
                }
            }
        }
        
        stage('Deploy') {
            steps {
                sh 'docker rm -f cinema-app || true'
                sh 'docker run -d --name cinema-app -p 8081:8080 $DOCKER_IMAGE:latest'
            }
        }
    }
    
    post {
        always {
            sh 'docker logout || true'
        }
    }
}
