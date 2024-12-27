pipeline {
    agent any
    environment {
        BRANCH = "${env.BRANCH_NAME}" // Use Jenkins' branch environment variable
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
                sh 'dotnet publish --configuration Release -o publish'
            }
        }
        stage('Deploy') {
            steps {
                echo "Deploying to Nginx server"
                script {
                    def publishPath = "${WORKSPACE}/publish"
                    sh """
                    sudo rm -rf /var/www/CMBackend/* &&
                    sudo cp -r ${publishPath}/* /var/www/CMBackend/ &&
                    sudo systemctl restart nginx
                    """
                }
            }
        }
    }
    post {
        success {
            echo "Build and deployment successful!"
        }
        failure {
            echo "Build or deployment failed."
        }
    }
}
