pipeline {
    agent any

    environment {
        SONAR_TOKEN = credentials('sonar-token')
        SONAR_HOST_URL = 'http://20.37.160.232:9000'
        SONAR_PROJECT_KEY = 'teclado'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh '''
                        sonar-scanner \
                        -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                        -Dsonar.sources=. \
                        -Dsonar.host.url=${SONAR_HOST_URL} \
                        -Dsonar.login=${SONAR_TOKEN}
                    '''
                }
            }
        }
    }
    post {
        always {
            echo 'Pipeline finished'
        }
        failure {
            echo 'Pipeline failed'
        }
        success {
            echo 'Pipeline succeeded'
        }
    }
}
