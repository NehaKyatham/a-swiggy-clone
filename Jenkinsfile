pipeline {
    agent any

    tools {
        jdk 'jdk17'
        nodejs 'node16'
    }

    environment {
        SCANNER_HOME = tool 'SonarQube-Scanner'
    }

    stages {

        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout from Git') {
            steps {
                git branch: 'main', url: 'https://github.com/NehaKyatham/a-swiggy-clone'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube-Servers') {
                    sh """
                        ${SCANNER_HOME}/bin/sonar-scanner \
                        -Dsonar.projectName=Swiggy-CI \
                        -Dsonar.projectKey=Swiggy-CI
                    """
                }
            }
        }

        stage('Quality Gate') {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'SonarQube-Token'
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }

        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }

        stage('Docker Build & Push') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'dockerhub-token', toolName: 'docker') {
                        sh "docker build -t swiggy-clone ."
                        sh "docker tag swiggy-clone nehakyatham/swiggy-clone:latest"
                        sh "docker push nehakyatham/swiggy-clone:latest"
                    }
                }
            }
        }

        stage('TRIVY Image Scan') {
            steps {
                sh "trivy image nehakyatham/swiggy-clone:latest > trivyimage.txt"
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    dir('Kubernetes') {
                        kubeconfig(credentialsId: 'kubernetes', serverUrl: '') {
                            sh "kubectl delete --all pods"
                            sh "kubectl apply -f deployment.yml"
                            sh "kubectl apply -f service.yml"
                        }
                    }
                }
            }
        }

    }
}

