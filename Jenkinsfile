pipeline {
    agent { label 'wsl' }

    environment {
        HOME = "${WORKSPACE}"             // Always use Jenkins workspace path
        MLFLOW_TRACKING_URI = 'http://127.0.0.1:5000'
    }

    stages {
        stage('Checkout Code') {
            steps {
                // Jenkins already checks out code automatically before stages
                // so this step can be optional
                git branch: 'main', url: 'https://github.com/yadneeka/mlops-mini.git'
            }
        }

        stage('Setup Python Env') {
            steps {
                sh '''
                cd $WORKSPACE
                python3 -m venv .venv
                . .venv/bin/activate
                pip install --upgrade pip
                pip install mlflow scikit-learn
                mkdir -p mlruns
                nohup mlflow server --backend-store-uri ./mlruns \
                    --default-artifact-root ./mlruns \
                    --host 127.0.0.1 --port 5000 > mlflow.log 2>&1 &
                sleep 5
                '''
            }
        }

        stage('Run MLflow Experiment') {
            steps {
                sh '''
                cd $WORKSPACE
                . .venv/bin/activate
                python3 mlflow_exp.py
                '''
            }
        }

        stage('Deploy to Minikube') {
            steps {
                sh '''
                cd $WORKSPACE
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
