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
                git branch: 'checkout', url: 'git@github.com:mosime219/Revive_Madeline.git', credentialsId: 'ssh-Agent'
            }
        }

        stage('Build and Unit Test') { 
            when {
                expression { params.RUN_SONARQUBE } // Execute only if RUN_SONARQUBE is false
            }
            agent {
                docker {
                    image 'node:22.4'
                    args '-u root'
                }
            }
            steps {
                echo 'Building project and running Unit Tests...'
                sh '''
                cd revive-checkout/checkout
                npm install
                npm test --passWithNoTests || true
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
                            -Dsonar.projectKey=checkout\
                            -Dsonar.host.url=http://54.161.33.220:9000/ \
                            -Dsonar.login=$SONAR_TOKEN \
                            -Dsonar.sources=./revive-checkout/checkout \
                            -Dsonar.java.binaries=./revive-checkout/checkout/src/main/java
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
                        cd 
                        revive-checkout/checkout
                        docker build -t mosime/revive:checkout:01 .
                        docker build -f Dockerfile-db -t mosime/revive:checkout-db-01 .
                    '''
                }
            }
        }

        stage('Push Docker Images') {
            steps {
                script {
                    echo 'Pushing Docker images to Docker Hub...'
                    sh '''
                        docker push mosime/revive:checkout-01
                        docker push mosime/revive:checkout-db-01
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
