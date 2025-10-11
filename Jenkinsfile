pipeline {
    agent any

    environment {
        // Variables configuradas automáticamente por JCasC
        SONAR_HOST_URL = "${env.SONARQUBE_URL}"
        SONAR_PROJECT_KEY = 'teclado-project'
        SONAR_PROJECT_NAME = 'Teclado - Typing Practice App'
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Clonando código fuente...'
                checkout scm
            }
        }

        stage('Verificar Herramientas') {
            steps {
                echo 'Verificando herramientas instaladas...'
                sh '''
                    echo "Node.js: $(node --version)"
                    echo "NPM: $(npm --version)"
                    echo "SonarScanner: $(sonar-scanner --version | head -1)"
                '''
            }
        }

        stage('Análisis de Código con SonarQube') {
            steps {
                echo 'Ejecutando análisis estático de código...'
                withCredentials([string(credentialsId: 'SONAR_TOKEN', variable: 'SONAR_TOKEN')]) {
                    sh """
                        sonar-scanner \
                          -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                          -Dsonar.projectName='${SONAR_PROJECT_NAME}' \
                          -Dsonar.sources=. \
                          -Dsonar.exclusions=css/**,**/*.css \
                          -Dsonar.javascript.file.suffixes=.js \
                          -Dsonar.host.url=${SONAR_HOST_URL} \
                          -Dsonar.token=${SONAR_TOKEN}
                    """
                }
            }
        }

        stage('Validación de Archivos') {
            steps {
                echo 'Verificando estructura del proyecto...'
                sh '''
                    echo "Archivos del proyecto:"
                    ls -lah
                    echo ""
                    echo "Validando archivos críticos:"
                    test -f index.html && echo "✓ index.html encontrado" || exit 1
                    test -f script.js && echo "✓ script.js encontrado" || exit 1
                    test -d css && echo "✓ carpeta css encontrada" || exit 1
                '''
            }
        }
    }

    post {
        success {
            echo '✅ ¡Pipeline ejecutado exitosamente!'
            echo '📊 Revisa el análisis en: http://20.12.193.55:9000/dashboard?id=${SONAR_PROJECT_KEY}'
        }
        failure {
            echo '❌ Pipeline falló - revisar logs'
        }
        always {
            echo '🧹 Pipeline finalizado'
        }
    }
}
