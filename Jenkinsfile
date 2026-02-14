pipeline {
    agent {
        docker {
            image 'python:3.10-slim'
            args '-u root'
        }
    }

    environment {
        DOCKERHUB_CREDS = credentials('dockerhub-creds')
        BEST_ACCURACY   = credentials('best-accuracy')
        IMAGE_NAME      = 'yourdockerhubusername/ml-model'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Setup Python Virtual Environment') {
            steps {
                sh '''
                python3 -m venv venv
                . venv/bin/activate
                pip install --upgrade pip
                pip install -r requirements.txt
                '''
            }
        }

        stage('Train Model') {
            steps {
                    sh '''
                    . venv/bin/activate
                    python scripts/train.py

                    mkdir -p app/artifacts
                    cp output/results/results.json app/artifacts/metrics.json
                    '''
            }
        }

        stage('Read Accuracy') {
            steps {
                script {
                    def metrics = readJSON file: 'app/artifacts/metrics.json'
                    env.CURRENT_ACCURACY = metrics.accuracy.toString()
                    echo "Current Accuracy: ${env.CURRENT_ACCURACY}"
                }
            }
        }

        stage('Compare Accuracy') {
            steps {
                script {
                    env.IS_BETTER = 'false'
                    if (env.CURRENT_ACCURACY.toFloat() > BEST_ACCURACY.toFloat()) {
                        env.IS_BETTER = 'true'
                        echo 'New model is better'
                    } else {
                        echo 'New model is NOT better'
                    }
                }
            }
        }

        stage('Build Docker Image') {
            sh '''
        echo $DOCKERHUB_CREDS_PSW | docker login \
        -u $DOCKERHUB_CREDS_USR --password-stdin

        docker build -t 2022bcs0187sujal/wine-quality:latest .
    '''
        }

        stage('Push Docker Image') {
            sh 'docker push 2022bcs0187sujal/wine-quality:latest'
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'app/artifacts/**', fingerprint: true
        }
    }
}
