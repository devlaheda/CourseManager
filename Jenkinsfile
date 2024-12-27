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
                sh 'dotnet publish --configuration Release -o publish'
            }
        }
        stage('Run App') {
            steps {
                echo "Starting the application"
                script {
                    def publishPath = "${WORKSPACE}/publish"
                    sh """
                    if [ -f app.pid ]; then
                        kill \$(cat app.pid) || true
                        rm app.pid
                    fi
                    nohup dotnet ${publishPath}/CourseManager.API.dll --urls=http://0.0.0.0:${APP_PORT} > app.log 2>&1 &
                    echo \$! > app.pid
                    """
                }
            }
        }
        stage('Configure Nginx') {
            steps {
                echo "Configuring Nginx"
                script {
                    sh """
                    sudo rm -rf /var/www/CMBackend/*
                    sudo cp -r ${WORKSPACE}/publish/* /var/www/CMBackend/
                    sudo systemctl restart nginx
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
