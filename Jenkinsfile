pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "valam75/sonarqube"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/valam75/jenkins-sonarqube-docker-demo.git'
            }
        }

        stage('Build') {
            steps {
                bat 'pip install -r requirements.txt'
            }
        }

        stage('Test') {
            steps {
                bat 'pytest'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarserver') {
                    bat """
                    sonar-scanner ^
                    -Dsonar.projectKey=demo ^
                    -Dsonar.sources=. ^
                    -Dsonar.host.url=http://34.227.173.238:9000 ^
                    -Dsonar.login=YOUR_TOKEN
                    """
                }
            }
        }

        stage('Docker Build') {
            steps {
                bat 'docker build -t %DOCKER_IMAGE% .'
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'Dockerlogin',
                usernameVariable: 'USER',
                passwordVariable: 'PASS')]) {

                    bat """
                    echo %PASS% | docker login -u %USER% --password-stdin
                    docker push %DOCKER_IMAGE%
                    """
                }
            }
        }

        stage('Deploy Container') {
            steps {
                bat 'docker run -d -p 5000:5000 %DOCKER_IMAGE%'
            }
        }

    }
}
