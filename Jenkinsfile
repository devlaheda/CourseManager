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
        stage('Publish') {
            steps {
                echo "Publishing the application"
                sh """
                rm -rf /var/www/CMBackend/ &
                dotnet publish --configuration Release --property:PublishDir=${WORKSPACE}/publish/
                """
            }
        }
        
        stage('Configure Nginx') {
            steps {
                echo "Configuring Nginx"
                script {
                    sh """
                    mv -f ${WORKSPACE}/publish/* /var/www/CMBackend/
                    """
                }
            }
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
