pipeline {
    agent any
    triggers {
        githubPush()  
        pollSCM('* * * * *')
    }
    environment {
        Instance = 'ubuntu@ec2-51-20-143-99.eu-north-1.compute.amazonaws.com'
    }
    stages {
        stage('Github Repo') {
            steps {
                echo 'Pulling the project from Github...'
                git changelog: false, poll: false, url: 'https://github.com/the-one-rvs/elevate-2.git'
            }
        }

        stage('DockerImage Create') {
            steps {
                script {
                    echo 'Building Docker Image...'
                    def buildNumber = env.BUILD_NUMBER
                    sh "docker build -t quasarcelestio/task-freelancer:0.1.${buildNumber} app"
                }
            }
        }

        stage('Trivy Scan') {
            steps {
                script {
                    echo 'Running Trivy Scan...'
                    def buildNumber = env.BUILD_NUMBER
                    sh "trivy image --format json --output trivy-report.json quasarcelestio/task-freelance:0.1.${buildNumber}"
                }
            }
        }

        stage('DockerImage Push') {
            steps {
                echo 'Pushing Docker Image to DockerHub...'
                script {
                    def buildNumber = env.BUILD_NUMBER
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhubcred') {
                        sh "docker push quasarcelestio/task-freelancer:0.1.${buildNumber}"
                    }
                }
            }
        }
    }
    post {
        always {
            echo 'Removing Docker Image from local...'
            script {
                def buildNumber = env.BUILD_NUMBER
                sh "docker rmi quasarcelestio/devsecops:0.1.${buildNumber}"
            }
        }
    }
}