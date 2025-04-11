pipeline {
    agent any

    environment {
        NVD_API_KEY = credentials('nvd-api-key')
    }
    
    stages {
        stage('SCA with OWASP Dependency Check') {
            steps {
                withCredentials([string(credentialsId: 'nvd-api-key', variable: 'NVD_API_KEY')]) {
                    dependencyCheck additionalArguments: "--nvdApiKey ${NVD_API_KEY}", odcInstallation: 'DP-Check'
            }
        }
    }

        stage('SonarQube Analysis') {
            steps {
                script {
                    scannerHome = tool 'SonarScanner'
                }
                withSonarQubeEnv('Sonarqube Server') {
                    sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=newsread-microservice-application"
                }
            }
        }

        stage('Build Docker Images') {
            steps {
                script {
                    sh 'docker build -t naveenjangid22/newsread-customize customize-service/'
                    sh 'docker build -t naveenjangid22/newsread-news news-service/'
                }
            }
        }

        stage('Containerize And Test') {
            steps {
                script {
                    sh 'docker run -d --name customize-service -e FLASK_APP=run.py naveenjangid22/newsread-customize && sleep 10 && docker logs customize-service && docker stop customize-service'
                    sh 'docker run -d --name news-service -e FLASK_APP=run.py naveenjangid22/newsread-news && sleep 10 && docker logs news-service && docker stop news-service'
                }
            }
        }

        stage('Push Images To Dockerhub') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'DockerHubPass', variable: 'DockerHubPass')]) {
                        sh 'docker login -u naveenjangid22 --password ${DockerHubPass}'
                    }
                    sh 'docker push naveenjangid22/newsread-news && docker push naveenjangid22/newsread-customize'
                }
            }
        }

        // Optional: Uncomment if you want Trivy scanning
        /*
        stage('Trivy scan on Docker images') {
            steps {
                sh 'TMPDIR=/home/jenkins'
                sh 'trivy image naveenjangid22/newsread-news:latest'
                sh 'trivy image naveenjangid22/newsread-customize:latest'
            }
        }
        */
    }

    post {
        always {
            sh 'docker rm news-service || true'
            sh 'docker rm customize-service || true'
        }
        success {
            sh 'docker logout'
        }
    }
}
