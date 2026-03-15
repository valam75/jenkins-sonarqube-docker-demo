pipeline {
    agent any

    tools {
        sonarRunner 'sonar-scanner'
    }

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
                sh '''
                python3 -m venv venv
                . venv/bin/activate
                pip install --upgrade pip
                pip install -r requirements.txt
                '''
            }
        }

        stage('Test') {
            steps {
                sh '''
                . venv/bin/activate
                pytest
                '''
            }
        }

        stage('SonarQube Analysis') {
    steps {
        withSonarQubeEnv('sonarserver') {
            sh '''
            sonar-scanner \
            -Dsonar.projectKey=demo \
            -Dsonar.sources=. \
            -Dsonar.host.url=http://184.72.110.102:9000
            '''
        }
    }
}

        stage('Quality Gate') {
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
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
