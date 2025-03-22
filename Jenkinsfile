pipeline {
    agent any

    parameters {
        booleanParam(name: 'RUN_SONARQUBE', defaultValue: false, description: 'Run SonarQube analysis?')
    }

    environment {
        SONAR_TOKEN = credentials('sonar-token') // Sonar token
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'cart', url: 'git@github.com:mosime219/Revive_Madeline.git', credentialsId: 'ssh-Agent'
            }
        }

        stage('Build and Unit Test') { 
            when {
                expression { params.RUN_SONARQUBE } // Execute only if RUN_SONARQUBE is false
            }
            agent {
                docker {
                    image 'maven:3.8.7-openjdk-18'
                    args '-u root'
                }
            }
            steps {
                echo 'Building project and running Unit Tests...'
                sh '''
                cd revive-cart/cart
                mvn clean compile
                mvn test  -DskipTests
                '''
            }
        }

        stage('SonarQube Analysis') {
            when {
                expression { params.RUN_SONARQUBE } // Execute only if RUN_SONARQUBE is true
            }
            environment {
                SCANNER_HOME = tool 'sonar' // Define the SonarQube scanner tool
            }
            steps {
                script {
                    // Perform SonarQube analysis using the SonarQube plugin
                    withSonarQubeEnv('sonar') { // 'Sonar' is the SonarQube server configured in Jenkins
                        sh """
                            ${SCANNER_HOME}/bin/sonar-scanner \
                            -Dsonar.projectKey=cart-microserve \
                            -Dsonar.host.url=http://3.91.249.70:9000/ \
                            -Dsonar.login=$SONAR_TOKEN \
                            -Dsonar.sources=./revive-cart/cart \
                            -Dsonar.java.binaries=./revive-cart/cart/src/main/java
                        """
                    }
                }
            }
        }

        stage('Docker Hub Login') {
            steps {
                script {
                    echo 'Logging into Docker Hub...'
                    withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DOCKER_HUB_USER', passwordVariable: 'DOCKER_HUB_PASS')]) {
                        sh "echo ${DOCKER_HUB_PASS} | docker login -u ${DOCKER_HUB_USER} --password-stdin"
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    echo 'Building Docker image...'
                    sh '''
                        cd revive-cart/cart
                        docker build -t mosime/revive:cart-01 .
                        docker build -f Dockerfile-dynamodb -t mosime/revive:cart-db-01 .
                        
                    '''
                }
            }
        }

        stage('Push Docker Images') {
            steps {
                script {
                    echo 'Pushing Docker images to Docker Hub...'
                    sh '''
                        docker push mosime/revive:cart-01
                        docker push mosime/revive:cart-db-01
                        
                    '''
                }
            }
        }

        stage('Clean Workspace') {
            steps {
                script {
                    echo 'Cleaning up the workspace...'
                    cleanWs() // Clean the workspace at the end
                }
            }
        }
    }

    post {
        always {
            script {
                cleanWs() // Clean the workspace at the end
            }
        }
    }
}