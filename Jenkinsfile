pipeline {
  agent { label 'linux' }            // run on your WSL agent

  environment {
    VENV = ".venv"
    MLFLOW_PORT = "5000"
    MLFLOW_URI = "http://127.0.0.1:${MLFLOW_PORT}"
    K8S_MANIFEST = "k8s-deploy.yml"
  }

  options {
    timeout(time: 30, unit: 'MINUTES')
    ansiColor('xterm')
  }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Prepare Python env') {
      steps {
        sh '''
          set -e
          # create venv (idempotent)
          python3 -m venv ${VENV} || true
          . ${VENV}/bin/activate
          pip install --upgrade pip
          pip install -r requirements.txt || pip install mlflow scikit-learn numpy
        '''
      }
    }

    stage('Start MLflow server (background)') {
      steps {
        // start MLflow if not already running on port
        sh '''
          set -e
          . ${VENV}/bin/activate
          if ss -ltn | grep -q ":${MLFLOW_PORT} "; then
            echo "MLflow already listening on ${MLFLOW_PORT}"
          else
            mkdir -p mlruns
            nohup mlflow server --backend-store-uri ./mlruns --default-artifact-root ./mlruns --host 127.0.0.1 --port ${MLFLOW_PORT} > mlflow-server.log 2>&1 &
            sleep 2
            echo "Started MLflow server (logs: mlflow-server.log)"
          fi
        '''
      }
    }

    stage('Run MLflow experiment') {
      steps {
        sh '''
          set -e
          . ${VENV}/bin/activate
          export MLFLOW_TRACKING_URI=${MLFLOW_URI}
          python3 mlflow_exp.py
        '''
      }
    }

    stage('Deploy to Kubernetes (via kubectl)') {
      steps {
        sh '''
          set -e
          # ensure minikube is running
          if minikube status | grep -q 'host: Running'; then
             echo "minikube running"
          else
             echo "starting minikube..."
             minikube start --driver=docker
          fi

          # apply k8s manifest
          kubectl apply -f ${K8S_MANIFEST}

          # wait for rollout (if it's a Deployment)
          if kubectl get deploy --ignore-not-found | grep -q $(basename ${K8S_MANIFEST} .yml | sed 's/-deploy//g'); then
            # try rollouts status for a known name (safe fallback below)
            sleep 2
          fi

          kubectl get pods -o wide
          kubectl get svc -o wide
        '''
      }
    }

    stage('Optional: Run Ansible playbook') {
      steps {
        sh '''
          set -e
          if [ -f ansible/deploy.yml ]; then
            ansible-playbook ansible/deploy.yml -i localhost, --connection=local
          else
            echo "No ansible/deploy.yml found — skipping"
          fi
        '''
      }
    }
  }

  post {
    success {
      echo "Pipeline finished SUCCESS"
    }
    failure {
      echo "Pipeline FAILED - check console output"
      sh 'tail -n 200 mlflow-server.log || true'
    }
  }
}
