pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "valam75/sonarqube:latest"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/valam75/jenkins-sonarqube-docker-demo.git'
            }
        }

        stage('Build') {
            steps {
                sh 'python3 -m pip install --upgrade pip'
                sh 'pip3 install -r requirements.txt'
            }
        }

        stage('Test') {
            steps {
                sh 'pytest'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarserver') {
                    sh '''
                    sonar-scanner \
                    -Dsonar.projectKey=demo \
                    -Dsonar.sources=. \
                    -Dsonar.host.url=http://34.227.173.238:9000 \
                    -Dsonar.login=$SONAR_AUTH_TOKEN
                    '''
                }
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE .'
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'Dockerlogin',
                usernameVariable: 'USER',
                passwordVariable: 'PASS')]) {

                    sh '''
                    echo $PASS | docker login -u $USER --password-stdin
                    docker push $DOCKER_IMAGE
                    '''
                }
            }
        }

        stage('Deploy Container') {
            steps {
                sh '''
                docker stop demo-container || true
                docker rm demo-container || true
                docker run -d -p 5000:5000 --name demo-container $DOCKER_IMAGE
                '''
            }
        }

    }
}
