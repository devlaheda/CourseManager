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
                dotnet publish --configuration Release --property:PublishDir=${WORKSPACE}/publish/
                """
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    withCredentials([string(credentialsId: '9ff08a91-a5de-490e-87d5-8af384719822', variable: 'SONAR_TOKEN')]) {
                        sh '''
                             ~/.dotnet/tools/dotnet-sonarscanner --version
                             ~/.dotnet/tools/dotnet-sonarscanner begin /k:"CMBack" \
                                /d:sonar.host.url="${SONARQUBE_URL}" \
                                /d:sonar.branch.name="${BRANCH}" \
                                /d:sonar.token="${SONAR_TOKEN}"
                            dotnet build
                             ~/.dotnet/tools/dotnet-sonarscanner end /d:sonar.token="${SONAR_TOKEN}"
                        '''
                    }
                }
            }
        }        
        stage('Configure Nginx') {
            steps {
                echo "Configuring Nginx"
                script {
                    sh """                    
                    rm -rf /var/www/CMBackend/ &
                    mv -f ${WORKSPACE}/publish/* /var/www/CMBackend/
                    """
                }
            }
        }
    }
    post {
        success {
            echo "Build, deployment, and application start were successful!"
            sh 'sudo systemctl restart kestrel-CMBackend.service' // this is not good practice but i need it in my home lab
        }
        failure {
            echo "Build, deployment, or application start failed."
        }
    }
}
