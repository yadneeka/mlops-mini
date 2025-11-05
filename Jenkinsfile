pipeline {
    agent { label 'wsl' }

    environment {
        MLFLOW_TRACKING_URI = "http://127.0.0.1:5000"
    }

    stage('Checkout Code') {
    steps {
        git branch: 'main',
            url: 'https://github.com/yadneeka/mlops-mini.git',
            changelog: false,
            poll: false,
            gitTool: 'linux-git'
    }
}


        stage('Setup Python Env') {
            steps {
                sh '''
                python3 -m venv venv
                . venv/bin/activate
                pip install --upgrade pip
                pip install mlflow scikit-learn
                '''
            }
        }

        stage('Run MLflow Experiment') {
            steps {
                sh '''
                . venv/bin/activate
                python3 mlflow_exp.py
                '''
            }
        }

        stage('Deploy to Minikube') {
            steps {
                sh '''
                echo "Applying K8s manifest to Minikube..."
                kubectl apply -f k8s-deploy.yml
                kubectl get pods
                '''
            }
        }
    }

    post {
        success {
            echo '✅ Pipeline executed successfully!'
        }
        failure {
            echo '❌ Pipeline failed!'
        }
    }
}

