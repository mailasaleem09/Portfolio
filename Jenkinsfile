pipeline {
    agent any

    environment {
        REPO_URL      = 'https://github.com/mailasaleem09/Portfolio.git'
        SONARQUBE_ENV = 'SonarQube-Server' // Jenkins System settings wala naam yahan likhein
        DOCKER_SERVER = 'ubuntu@ip-172-31-18-221'
    }

    stages {
        stage('Checkout') {
            steps {
                // Purana data saaf karne ke liye cleanWs use hota hai
                cleanWs()
                git branch: 'master', url: "${REPO_URL}"
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    // Tool ka naam "SonarQube Scanner" hona chahiye (Check Global Tool Configuration)
                    def scannerHome = tool 'SonarQube Scanner'
                    withSonarQubeEnv("${SONARQUBE_ENV}") {
                        sh """
                        ${scannerHome}/bin/sonar-scanner \
                        -Dsonar.projectKey=portfolio-cloud \
                        -Dsonar.sources=index.html \
                        -Dsonar.qualitygate.wait=false
                        """
                    }
                }
            }
        }

        stage('Docker Build & Deploy') {
            steps {
                // 'docker-credentials' ID Jenkins mein SSH Username with Private Key honi chahiye
                sshagent(['docker-credentials']) {
                    sh """
                    # Sab files server par bhejne ke liye
                    scp -o StrictHostKeyChecking=no index.html Dockerfile ${DOCKER_SERVER}:/home/ubuntu/

                    # Remote server par commands chalane ke liye
                    ssh -o StrictHostKeyChecking=no ${DOCKER_SERVER} "
                        cd /home/ubuntu
                        sudo docker build -t portfolio-app .
                        sudo docker stop portfolio-app || true
                        sudo docker rm portfolio-app || true
                        sudo docker run -d -p 80:80 --name portfolio-app portfolio-app
                    "
                    """
                }
            }
        }
    }

    post {
        success {
            echo "Deployment Successful! URL: http://172.31.18.221"
        }
        failure {
            echo "Pipeline Failed. Please check the logs."
        }
    }
}