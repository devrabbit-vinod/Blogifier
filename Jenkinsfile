pipeline {
    agent any
    
    environment {
        DOTNET_CLI_TELEMETRY_OPTOUT = '1'
        SOLUTION_FILE = 'Blogifier.sln'
        PROJECT_FILE = 'src/Blogifier/Blogifier.csproj'
        BUILD_CONFIGURATION = 'Release'
        PUBLISH_DIR = 'publish'
        HOST_PORT = '9012'
    }
  
    stages {
        stage('Checkout') {
            steps {
                checkout scm
                script {
                    currentBuild.displayName = "#${BUILD_NUMBER} - ${env.GIT_BRANCH}"
                    currentBuild.description = "Commit: ${env.GIT_COMMIT.take(8)}"
                }
            }
        }
        
        stage('Restore Dependencies') {
            steps {
                sh "dotnet restore ${SOLUTION_FILE}"
            }
        }
        
        stage('Build') {
            steps {
                sh """
                dotnet build ${SOLUTION_FILE} \
                --configuration ${BUILD_CONFIGURATION} \
                --no-restore \
                /p:Version=${BUILD_NUMBER}
                """
            }
        }
      
        stage('Security Scan') {
            steps {
                sh 'dotnet list package --vulnerable'
            }
        }
        
        stage('Publish') {
            steps {
                sh """
                dotnet publish ${PROJECT_FILE} \
                -c ${BUILD_CONFIGURATION} \
                -o ${PUBLISH_DIR} \
                --no-restore
                """
            }
        }
        
        stage('Host Application') {
            steps {
                script {
                    // Stop any existing running application
                    sh "sudo fuser -k ${HOST_PORT}/tcp || true"
                    
                    // Run the application
                    sh """
                    nohup dotnet ${PUBLISH_DIR}/${PROJECT_FILE.split('/').last().replace('.csproj', '.dll')} \
                    --urls "http://*:${HOST_PORT}" > ${PUBLISH_DIR}/app.log 2>&1 &
                    """
                    
                    // Verify application is running
                    sh "sleep 5"
                    sh "curl -I http://localhost:${HOST_PORT}"
                }
            }
        }
    }
    
    post {
        always {
            archiveArtifacts artifacts: "${PUBLISH_DIR}/**"
        }
    }
}
