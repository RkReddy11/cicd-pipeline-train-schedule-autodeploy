pipeline {
    agent any

    environment {
        DOCKER_IMAGE_NAME = "rkreddy12/train-schedule-autodeploy"
        KUBECONFIG_PATH = "/home/rkreddy/config"
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
                    def app = docker.build(DOCKER_IMAGE_NAME)
                    app.inside {
                        sh 'echo Hello, World!'
                    }
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    def app = docker.build(DOCKER_IMAGE_NAME)
                    docker.withRegistry('https://registry.hub.docker.com', 'docker_hub_login') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push('latest')
                    }
                }
            }
        }
        
        stage('Check Kubernetes Config') {
    steps {
        sh 'cat /home/rkreddy/config'
    }
}
        
stage('Canary Deploy') {
            environment {
                CANARY_REPLICAS = 1
            }
            steps {
                sh "kubectl --kubeconfig=${env.KUBECONFIG_PATH} apply -f train-schedule-kube-canary.yml"
            }
        }

        stage('Deploy to Production') {
            environment {
                CANARY_REPLICAS = 0
            }
            steps {
                input 'Deploy to Production?'
                milestone(1)
                sh "kubectl --kubeconfig=${env.KUBECONFIG_PATH} apply -f train-schedule-kube-canary.yml"
                sh "kubectl --kubeconfig=${env.KUBECONFIG_PATH} apply -f train-schedule-kube.yml"
            }
        }
    }
}
