pipeline {
    agent { label 'wsl' }

    environment {
        MLFLOW_TRACKING_URI = 'http://127.0.0.1:5000'
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
                cd ~/mlops-mini
                python3 -m venv .venv
                . .venv/bin/activate
                pip install mlflow scikit-learn
                mkdir -p mlruns
                nohup mlflow server --backend-store-uri ./mlruns --default-artifact-root ./mlruns --host 127.0.0.1 --port 5000 > mlflow.log 2>&1 &
                '''
            }
        }

        stage('Run MLflow Experiment') {
            steps {
                sh '''
                cd ~/mlops-mini
                . .venv/bin/activate
                python3 mlflow_exp.py
                '''
            }
        }

        stage('Deploy to Minikube') {
            steps {
                sh '''
                cd ~/mlops-mini
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
