pipeline {
    agent any

    environment {
        REPO_URL      = 'https://github.com/mailasaleem09/Portfolio'
        SONARQUBE_ENV = 'SonarQube-Server'
        DOCKER_SERVER = 'ubuntu@ip-172-31-18-221'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'master', url: "${REPO_URL}"
            }
        }

        stage('Quality & Deployment') {
            parallel {
                stage('SonarQube Analysis') {
                    steps {
                        script {
                            def scannerHome = tool 'SonarQube Scanner'
                            withSonarQubeEnv("${SONARQUBE_ENV}") {
                                // Background scan: results ka wait nahi karega
                                sh """
                                ${scannerHome}/bin/sonar-scanner \
                                -Dsonar.projectKey=portfolio-cloud \
                                -Dsonar.sources=. \
                                -Dsonar.exclusions=*/.css,*/.js,node_modules/** \
                                -Dsonar.qualitygate.wait=false
                                """
                            }
                        }
                    }
                }

                stage('Docker Build & Deploy') {
                    steps {
                        // Jenkins credentials ID 'ubuntu' honi chahiye
                        sshagent(['docker-credentials']) {
                            sh """
                            scp -o StrictHostKeyChecking=no index.html Dockerfile ${DOCKER_SERVER}:/home/ubuntu/

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
        }
    }
}