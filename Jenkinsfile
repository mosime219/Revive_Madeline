pipeline {
    agent any

    parameters {
        booleanParam(name: 'RUN_SONARQUBE', defaultValue: false, description: 'Run SonarQube analysis?')
    }

    environment {
        SONAR_TOKEN = credentials('sonar-token') // Sonar token
    }

    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs() // Clean the workspace at the beginning
            }
        }

        stage('Checkout') {
            steps {
                git branch: 'catalog', url: 'git@github.com:mosime219/Revive_Madeline.git', credentialsId: 'ssh-Agent'
            }
        }

        stage('Build and Unit Test') { 
            when {
                expression { params.RUN_SONARQUBE } // Execute only if RUN_SONARQUBE is false
            }
            agent {
                docker {
                    image 'golang:1.22.5'
                    args '-u root'
                }
            }
            steps {
                echo 'Building project and running Unit Tests...'
                sh '''
                    cd revive-catalog/catalog
                    go test
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
                    withSonarQubeEnv('sonar') { // 'sonar' is the SonarQube server configured in Jenkins
                        sh """
                            ${SCANNER_HOME}/bin/sonar-scanner \
                            -Dsonar.projectKey=Catalog-microserve \
                            -Dsonar.host.url=http://54.163.223.202:9000/ \
                            -Dsonar.login=$SONAR_TOKEN \
                            -Dsonar.sources=./revive-catalog/catalog
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
                        cd revive-catalog/catalog
                        docker build -t mosime/revive:catalog-01 .
                        docker build -f Dockerfile-db -t mosime/revive:catalog-db-01 .
                    '''
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    echo 'Pushing Docker image to Docker Hub...'
                    sh '''
                        docker push mosime/revive:catalog-01
                        docker push mosime/revive:catalog-db-01
                    '''
                }
            }
        }

        stage('Clean Docker') {
            steps {
                script {
                    echo 'Cleaning up unused Docker images and containers...'
                    sh '''
                        docker system prune -af
                    '''
                }
            }
        }
    }

    post {
        always {
            cleanWs() // Clean the workspace at the end
        }
    }
}