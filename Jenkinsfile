pipeline {
    pipeline {
    agent { label 'wsl' }

    environment {
        MLFLOW_TRACKING_URI = "http://127.0.0.1:5000"
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/yadneeka/mlops-mini.git'
            }
        }

        stage('Setup Python Env') {
            steps {
                sh '''
                python3 -m venv .venv
                . .venv/bin/activate
                pip install --upgrade pip
                pip install -r requirements.txt
                '''
            }
        }

        stage('Run MLflow Experiment') {
            steps {
                sh '''
                . .venv/bin/activate
                export MLFLOW_TRACKING_URI=${MLFLOW_TRACKING_URI}
                python3 mlflow_exp.py
                '''
            }
        }

        stage('Deploy to Minikube') {
            steps {
                sh '''
                echo "Deploying to Minikube..."
                kubectl apply -f k8s-deploy.yml
                kubectl get pods
                '''
            }
        }
    }

    post {
        success {
            echo '✅ Pipeline completed successfully!'
        }
        failure {
            echo '❌ Pipeline failed!'
        }
    }
}

