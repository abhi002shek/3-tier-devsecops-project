pipeline {
    agent any
    
    tools {
        nodejs 'nodejs23'
    }
    
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }
    
    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/YOUR_USERNAME/3-tier-devsecops-project.git'
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh '''
                        $SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectKey=NodeJS-Project \
                        -Dsonar.projectName=NodeJS-Project \
                        -Dsonar.sources=. \
                        -Dsonar.exclusions=node_modules/**,coverage/**,*.log
                    '''
                }
            }
        }
        
        stage('Quality Gate') {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
                }
            }
        }
        
        stage('Docker Build & Tag') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh "docker build -t abhi00shek/3tier-devsecops:latest ."
                    }
                }
            }
        }
        
        stage('Docker Push') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh "docker push abhi00shek/3tier-devsecops:latest"
                    }
                }
            }
        }
        
        stage('Approval') {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    input message: 'Approve deployment to PRODUCTION?', submitter: 'Abhishek'
                }
            }
        }
        
        stage('Deployment To Prod') {
            steps {
                script {
                    withKubeConfig(caCertificate: '', clusterName: 'devopsshack-cluster', contextName: '', credentialsId: 'k8-prod-token', namespace: 'prod', restrictKubeConfigAccess: false, serverUrl: 'https://F8BEA0C74986ECAED24C7AC39966425F.yl4.ap-south-2.eks.amazonaws.com') {
                        sh 'kubectl apply -f k8s-prod/sc.yaml'
                        sleep 20
                        sh 'kubectl apply -f k8s-prod/mysql.yaml -n prod'
                        sh 'kubectl apply -f k8s-prod/backend.yaml -n prod'
                        sh 'kubectl apply -f k8s-prod/frontend.yaml -n prod'
                        sh 'kubectl apply -f k8s-prod/ci.yaml'
                        sh 'kubectl apply -f k8s-prod/ingress.yaml -n prod'
                        sleep 30
                    }
                }
            }
        }
        
        stage('Verify Deployment To Prod') {
            steps {
                script {
                    withKubeConfig(caCertificate: '', clusterName: 'devopsshack-cluster', contextName: '', credentialsId: 'k8-prod-token', namespace: 'prod', restrictKubeConfigAccess: false, serverUrl: 'https://F8BEA0C74986ECAED24C7AC39966425F.yl4.ap-south-2.eks.amazonaws.com') {
                        sh 'kubectl get pods -n prod'
                        sleep 20
                        sh 'kubectl get ingress -n prod'
                    }
                }
            }
        }
    }
    
    post {
        always {
            cleanWs()
        }
        success {
            echo 'Pipeline executed successfully!'
        }
        failure {
            echo 'Pipeline failed. Check logs for details.'
        }
    }
}
