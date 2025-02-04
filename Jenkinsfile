pipeline {
    agent any
    environment {
        BRANCH = "${env.BRANCH_NAME}" // Use Jenkins' branch environment variable
        APP_PORT = "5000" // Define the port for the application to run
    }
    stages {
        stage('Checkout') {
            steps {
                echo "Pulling branch: ${BRANCH}"
                checkout scm // Default source control management configuration
            }
        }
        stage('Restore') {
            steps {
                echo "Restoring .NET dependencies"
                sh 'dotnet restore'
            }
        }
        stage('Build') {
            steps {
                echo "Building the project"
                sh 'dotnet build --configuration Release'
            }
        }
         stage('Test') {
            steps {
                echo "NO test for the moment"
            }
        }
        stage('Publish') {
            steps {
                echo "Publishing the application"
                sh("dotnet publish --configuration Release --property:PublishDir=${WORKSPACE}/publish/")
             
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    withCredentials([string(credentialsId: 'CMBackEndSec', variable: 'SONAR_TOKEN')]) {
                        sh('~/.dotnet/tools/dotnet-sonarscanner begin /k:"CMBackEnd"  /d:sonar.host.url="${SONAR_HOST_URL}" /d:sonar.branch.name="${BRANCH}" /d:sonar.token="${SONAR_TOKEN}"')
                        sh('dotnet build')
                        sh(' ~/.dotnet/tools/dotnet-sonarscanner end /d:sonar.token="${SONAR_TOKEN}"')
                    }
                }
            }
        }        
        stage('Container Hosting') {
            steps {
                echo "Configuring Nginx"
                script {
                    sh("docker build  -f .\CourseManager.API\Dockerfile -t aspnet-app .")
                    sh("docker stop aspnet-app || true")
                    sh("docker rm aspnet-app || true")
                    sh("docker run -d   --name aspnet-app   -p 5000:8080  --restart unless-stopped  aspnet-app")                    
                }
            }
        }
        stage('Verify') {
          steps {
            // Check if the container is running
            sh 'docker ps --filter "name=aspnet-app" --format "{{.Status}}" | grep Up'    
            // Optional: Smoke test the endpoint
            sh 'curl -I http://localhost:5000'
          }
    }
    post {
        success {
            echo "Build, deployment, and application start were successful!"
        }
        failure {
            echo "Build, deployment, or application start failed."
        }
    }
}
