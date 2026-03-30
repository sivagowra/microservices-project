pipeline {
    agent any
    
    environment {
        K8_CLUSTER_NAME = 'EKS-1'
        K8_CREDENTIALS_ID = 'k8-token'
        K8_SERVER_URL = 'https://97EFB5ECAEB87BA9B9EC7B775B12B51F.gr7.us-east-2.eks.amazonaws.com'
    }

    stages {
        stage('Prepare Branch Namespace') {
            steps {
                script {
                    // Keep namespace DNS-1123 compatible and deterministic per branch.
                    def rawBranch = env.BRANCH_NAME ?: 'main'
                    def safeBranch = rawBranch.toLowerCase().replaceAll(/[^a-z0-9-]/, '-')
                    env.DEPLOY_NAMESPACE = "webapps-${safeBranch}".take(63)
                }
                echo "Deploy namespace: ${env.DEPLOY_NAMESPACE}"
            }
        }

        stage('Deploy To Kubernetes') {
            steps {
                withKubeCredentials(kubectlCredentials: [[caCertificate: '', clusterName: "${env.K8_CLUSTER_NAME}", contextName: '', credentialsId: "${env.K8_CREDENTIALS_ID}", namespace: "${env.DEPLOY_NAMESPACE}", serverUrl: "${env.K8_SERVER_URL}"]]) {
                    sh "kubectl create namespace ${env.DEPLOY_NAMESPACE} --dry-run=client -o yaml | kubectl apply -f -"
                    sh "kubectl apply -f deployment-service.yml -n ${env.DEPLOY_NAMESPACE}"
                }
            }
        }
        
        stage('verify Deployment') {
            steps {
                withKubeCredentials(kubectlCredentials: [[caCertificate: '', clusterName: "${env.K8_CLUSTER_NAME}", contextName: '', credentialsId: "${env.K8_CREDENTIALS_ID}", namespace: "${env.DEPLOY_NAMESPACE}", serverUrl: "${env.K8_SERVER_URL}"]]) {
                    sh "kubectl get svc -n ${env.DEPLOY_NAMESPACE}"
                }
            }
        }
    }
}
