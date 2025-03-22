pipeline {
    agent any

    parameters {
        booleanParam(name: 'RUN_SONARQUBE', defaultValue: false, description: 'Run SonarQube analysis?')
    }

    environment {
        SONAR_TOKEN = credentials('sonar-token') // Sonar token
    }

    stages {
        stage('checkout') {
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
                cd revive-orders/orders
                mvn clean compile
                mvn test
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
                            -Dsonar.projectKey=Orders-microserve \
                            -Dsonar.host.url=http://3.91.249.70:9000/ \
                            -Dsonar.login=$SONAR_TOKEN \
                            -Dsonar.sources=./revive-orders/orders \
                            -Dsonar.java.binaries=./revive-Orders/orders/src/main/java
                        """
                    }
                }
            }
        }

        stage('Docker Hub Login') {
            steps {
                script {
                    echo 'Logging into Docker Hub...'
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-tgitech', usernameVariable: 'DOCKER_HUB_USER', passwordVariable: 'DOCKER_HUB_PASS')]) {
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
                        cd revive-Orders/orders
                        docker build -t tgitech/revive-orders:01 .
                        docker build -f Dockerfile-db -t tgitech/revive-orders:db-01 .
                        docker build -f Dockerfile-rabbit-mq -t tgitech/revive-orders:rabbit-mq-01 .
                    '''
                }
            }
        }

        stage('Push Docker Images') {
            steps {
                script {
                    echo 'Pushing Docker images to Docker Hub...'
                    sh '''
                        docker push tgitech/revive-orders:01
                        docker push tgitech/revive-orders:rabbit-mq-01
                        docker push tgitech/revive-orders:db-01
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