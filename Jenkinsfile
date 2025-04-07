pipeline {
    agent any
    
   environment {
        DOTNET_CLI_TELEMETRY_OPTOUT = '1'
        SOLUTION_FILE = 'Blogifier.sln'          // Change to your solution file
        PROJECT_FILE = 'src/Blogifier/Blogifier.csproj'  // Change to your project file
        BUILD_CONFIGURATION = 'Release'
        PUBLISH_DIR = 'publish'
        HOST_PORT = '9012'                         // Set your custom port here
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
                sh 'dotnet restore ${SOLUTION_FILE}'
            }
        }
        
        stage('Build') {
            steps {
                sh """
                dotnet build --configuration ${BUILD_CONFIGURATION} \
                --no-restore \
                /p:Version=${BUILD_NUMBER}
                """
            }
        }
      
        stage('Security Scan') {
            steps {
                // Example: Run security scanning tools
                sh 'dotnet list package --vulnerable'
                // Could add OWASP Dependency Check or other scanners
            }
        }
        
        stage('Publish Application') {
            steps {
                script {
                    // Stop any existing running application on this port
                    sh "sudo fuser -k ${HOST_PORT}/tcp || true"
                    
                    // Run the application in background with your custom port
                    sh """
                    nohup dotnet ${PUBLISH_DIR}/${PROJECT_FILE.split('/').last().replace('.csproj', '.dll')} \
                    --urls "http://*:${HOST_PORT}" > ${PUBLISH_DIR}/app.log 2>&1 &
                    """
                    
                    // Verify application is running
                    sh "sleep 5" // Wait for app to start
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
