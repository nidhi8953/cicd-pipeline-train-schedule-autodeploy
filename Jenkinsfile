pipeline {
    agent any
    environment {
        //be sure to replace "bhavukm" with your own Docker Hub username
        DOCKER_IMAGE_NAME = "nidhisarup8953/train-schedule"
    }
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    app = docker.build(DOCKER_IMAGE_NAME)
                    app.inside {
                        sh 'echo Hello, World!'
                    }
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker_hub_login') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
        }
        stage('CanaryDeploy') {
            steps {
                withKubeConfig([
                    credentialsId: 'kubeconfig',
                    serverUrl: 'https://192.168.49.2:8443'
                ]) {
                    sh 'kubectl apply -f train-schedule-kube-canary.yml'
                }

            }
        }  
        stage('DeployToProduction') {
            steps {
                input 'Deploy to Production?'
                milestone(1)
                withKubeConfig([
                    credentialsId: 'kubeconfig',
                    serverUrl: 'https://192.168.49.2:8443'
                ]) {
                    sh 'kubectl apply -f train-schedule-kube-canary.yml'
                }
                withKubeConfig([
                    credentialsId: 'kubeconfig',
                    serverUrl: 'https://192.168.49.2:8443'
                ]) {
                    sh 'kubectl apply -f train-schedule-kube.yml'
                }
              
            }
        }
    }
}
