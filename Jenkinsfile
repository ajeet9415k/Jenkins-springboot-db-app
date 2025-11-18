pipeline {
    agent any

    tools {
        maven 'maven'
        jdk 'jdk17'
    }

    stages {

        stage('checkout') {
            steps {
                git changelog: false,
                    credentialsId: 'github',
                    poll: false,
                    url: 'https://github.com/ajeet9415k/Jenkins-springboot-db-app'
            }
        }

        stage('build') {
            steps {
                sh 'mvn clean install'
            }
        }
        stage('docker build & push') {
            steps {
                script {
                    // FIX: ensure Jenkins uses correct Docker context
                    sh "docker context use default || true"
                    
                    // Use system Docker Desktop; do NOT use toolName
                    docker.withRegistry('https://index.docker.io/v1/', 'docker') {

                        // Build Docker image
                        sh "docker build -t jenkins-springboot-db-app ."

                        // Tag image for Docker Hub
                        sh "docker tag jenkins-springboot-db-app ajeet9415/jenkins-springboot-db-app:latest"

                        // Push image
                        sh "docker push ajeet9415/jenkins-springboot-db-app:latest"
                    }
                }
            }
        }
        
        stage("deploy to K8S"){
            steps{
                script{
                    withKubeConfig([credentialsId: 'kubeconfig-local']) {
                        sh "kubectl config use-context minikube || true"
                        sh """
                            kubectl apply -f mysql-configMap.yml
                            kubectl apply -f mysql-secrets.yml
                            kubectl apply -f db-deployment.yml
                            kubectl apply -f app-deployment.yml
                            kubectl rollout restart deployment/springboot-crud-deployment
                        """
                    }
                }
            }
        }
    }
}
