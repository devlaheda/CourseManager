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
                sh 'dotnet publish --configuration Release --property:PublishDir=${WORKSPACE}/publish/'
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
        stage('Run App') {
            steps {
                echo "Starting the application"
                script {
                    def publishPath = "${WORKSPACE}/publish"
                    sh """
                    if [ -f ~/app.pid ]; then
                        kill \$(cat ~/app.pid) || true
                        rm ~/app.pid
                    fi
                    nohup dotnet /var/www/CMBackend//CourseManager.API.dll --urls=http://127.0.0.1:${APP_PORT} > ~/app.log 2>&1 &
                    echo \$! > ~/app.pid
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
