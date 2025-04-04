pipeline {
    agent any

    parameters {
        booleanParam(name: 'RUN_SONARQUBE', defaultValue: true, description: 'Run SonarQube Analysis')
    }

    environment {
        SONAR_TOKEN = credentials('sonar-token') // Sonar token from Jenkins credentials
    }

    stages {
        stage('Cleanup Workspace') {
            steps {
                script {
                    cleanWs() // Clean the workspace at the beginning
                }
            }
        }

        stage('Checkout') {
            steps {
                git branch: 'asset', url: 'git@github.com:mosime219/Revive_Madeline.git', credentialsId: 'ssh-Agent'
            }
        }

        stage('SonarQube Analysis') {
            when {
                expression { !params.RUN_SONARQUBE } // Execute only if RUN_SONARQUBE is true
            }
            environment {
                SCANNER_HOME = tool 'sonar' // SonarQube scanner tool in Jenkins
            }
            steps {
                script {
                    // Perform SonarQube analysis using the SonarQube plugin
                    withSonarQubeEnv('sonar') { // 'sonar' is the SonarQube server configured in Jenkins
                        sh """
                            ${SCANNER_HOME}/bin/sonar-scanner \
                            -Dsonar.projectKey=Assets-microservice \
                            -Dsonar.host.url=http://98.81.192.201:9000/ \
                            -Dsonar.login=$SONAR_TOKEN \
                            -Dsonar.sources=./revive-asset/assets
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
                        cd revive-asset/assets
                        docker build -t mosime/revive:asset-01 .
                    '''
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    echo 'Pushing Docker image to Docker Hub...'
                    sh '''
                        docker push mosime/revive:asset-01
                    '''
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