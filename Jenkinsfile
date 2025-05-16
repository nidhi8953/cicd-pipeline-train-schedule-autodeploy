pipeline {
    agent any
    environment {
        //be sure to replace "bhavukm" with your own Docker Hub username
        DOCKER_IMAGE_NAME = "nidhisarup8953/train-schedule"
    }
    stages {
         stage('Checkout') {
            steps {
                checkout scm  // This is crucial for branch detection
                script {
                    // This will get the actual branch name
                    def BRANCH = sh(returnStdout: true, script: 'git rev-parse --abbrev-ref HEAD').trim()
                    echo "Current branch: ${BRANCH}"
                    
                    // Alternatively, if you want the remote tracking branch
                    def REMOTE_BRANCH = sh(returnStdout: true, script: 'git branch -r --contains HEAD | sed "s/origin\\///" | head -1').trim()
                    echo "Remote branch: ${REMOTE_BRANCH}"
                }
            }
        }
        stage('Build') {
            steps {

                sh 'rm -rf build/nodejs/'
                sh 'rm -rf build/npm/'
                sh 'rm -rf node_modules/'
                echo 'Running build automation'
                echo "Displayed*******************************************"
                sh 'chmod +x gradlew'
                echo "Displayed***************uu****************************"
                sh 'java -version'
                echo "Displayed*******************************************"
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('Debug Branch') {
            steps {
                script {
                    echo "Current branch: ${env.BRANCH_NAME}"
                }
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
                    docker.withRegistry('https://registry.hub.docker.com', 'docker-image') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
        }
        stage('Setup kubectl') {
            steps {
                sh '''
                    curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
                    chmod +x kubectl
                    mkdir -p ${HOME}/bin
                    mv kubectl ${HOME}/bin/
                    export PATH=${HOME}/bin:${PATH}
                '''
                // Verify installation
                sh 'kubectl version --client --short'
            }
        }
        stage('Deploy') {
            steps {                         
                // OR Method 2: Using text credentials if you have the kubeconfig content
                withCredentials([string(credentialsId: 'kubeconfig_id', variable: 'KUBECONFIG_CONTENT')]) {
                    sh '''
                        echo "$KUBECONFIG_CONTENT" > kubeconfig.yaml
                        export KUBECONFIG=kubeconfig.yaml
                        kubectl apply -f train-schedule-kube-canary.yaml
                        rm kubeconfig.yaml
                    '''
                }
            }
        }
        stage('CanaryDeploy') {

            environment { 
                CANARY_REPLICAS = 1
            }
            steps {
                kubernetesDeploy(
                    kubeconfigId: 'kubeconfig_id',
                    configs: 'train-schedule-kube-canary.yml',
                    enableConfigSubstitution: true
                )
            }
        }
        stage('DeployToProduction') {
          
            environment { 
                CANARY_REPLICAS = 0
            }
            steps {
                input 'Deploy to Production?'
                milestone(1)
                kubernetesDeploy(
                    kubeconfigId: 'kubeconfig_id',
                    configs: 'train-schedule-kube-canary.yml',
                    enableConfigSubstitution: true
                )
                kubernetesDeploy(
                    kubeconfigId: 'kubeconfig',
                    configs: 'train-schedule-kube.yml',
                    enableConfigSubstitution: true
                )
            }
        }
    }
}
