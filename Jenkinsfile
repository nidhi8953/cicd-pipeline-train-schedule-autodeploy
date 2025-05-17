pipeline {
    agent any
    environment {
        //be sure to replace "bhavukm" with your own Docker Hub username
        DOCKER_IMAGE_NAME = "nidhisarup8953/train-schedule"

        // Set PATH to include directory where we'll install kubectl
        PATH = "${env.HOME}/bin:${env.PATH}"

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
 	stage('Install Kubectl--'){
	    steps {
				sh '''
				curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
                		chmod +x kubectl
                		mv kubectl /usr/local/bin/kubectl
				'''
	    }
	}
	stage("k8-----") {
            steps {
                script {
                        kubeconfig(caCertificate: 'LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURCakNDQWU2Z0F3SUJBZ0lCQVRBTkJna3Foa2lHOXcwQkFRc0ZBREFWTVJNd0VRWURWUVFERXdwdGFXNXAKYTNWaVpVTkJNQjRYRFRJMU1EVXhOVEl3TWpRd04xb1hEVE0xTURVeE5ESXdNalF3TjFvd0ZURVRNQkVHQTFVRQpBeE1LYldsdWFXdDFZbVZEUVRDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBTW5GCmR5d1FMeFA0emFROSs3SDc3dHJHNk1SRndmQ1FIZy9aTEZXT1pGL0JoNzd3VmxsNFVRTlpDSm9Gckt1ZGxZQ1QKT2d3NE1QS0VCQTg5Qm04eHpraDdrQUowSURMNU1MS3ZiSGdFam8vblBIQncvaDVUSGhBMVZXelduMVh2bnZsTQpobExjMjhjNklNZVMwZWZCQVgzSnJ5Y3RNVEUxNGxEZHZsM3RXbWpQMm0xYkR3M3ZaOFVVTWdicENJVk0vSDBaCjVsbWo1dnkwK1FGcmlPS1cyanFhZVRldGt3UUc4cDd3V1UxTGgveW51S2djT3hiWnVua3pXbVBJZ00vVEhxdTUKV0RwMGMrcnl5WGlVZnZYa2FvajhCaEkzMzZXUUNQRGtzSm40TFByeWVCUkkzKytVYlNValArcUdMSVZZMUNXbQp5VkNWeXhRSEtuMDc4b2pEeUVjQ0F3RUFBYU5oTUY4d0RnWURWUjBQQVFIL0JBUURBZ0trTUIwR0ExVWRKUVFXCk1CUUdDQ3NHQVFVRkJ3TUNCZ2dyQmdFRkJRY0RBVEFQQmdOVkhSTUJBZjhFQlRBREFRSC9NQjBHQTFVZERnUVcKQkJUeHcwdlNyTTE3S3hnayt0SWJ5ZjhmQ3Q4eEtEQU5CZ2txaGtpRzl3MEJBUXNGQUFPQ0FRRUFPem5EQlNvawpVWTNHQ1dnV3hKMkhxdnpSMHp1VUVIa0VVc1FrWjU3WVlheERmUDFBTzJTZUZwZEw4T3RCdmRNOWxjdjhidFNxCkRJS2VyTkZHNEhucC9LNjZCdTZWQUtMeTBhYVMycnJrNjEvMCtHYU5DQm5pQkRLdTlnOEFSZGd3TjR2WGRscEEKRGU0OThRYi9qRmxPMDJMVEdxcEszeXFsR1JRalo4UStMRXpOMmlGeUo4TVBwcEIxSzhzdFdqRTVXZTI0a1V3cQpQYXVTczBtRmFZRS9xVC9YM0cybTRnYUZ1QXM0Z0dGVDNSTVY3OENEYVJWZVhWRGlZRnh3RHVhNWFnS0V4Q0o5Cm50c1RYN2ZPb0FDRUJYTDJUc0xSZFZrWjNSY0MwcFNGdWV4TWhMbFhzUVhnOUJzY3lmNzI3NE1tQ3QxLytSRFMKdEhNaW9lZkErWmxTOEE9PQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==', credentialsId: 'k-config', serverUrl: 'https://192.168.49.2:8443') {
                                sh 'kubectl get pods'
                        }
                }
            }
        }
    
    }
}
