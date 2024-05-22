pipeline {
    agent {
        label 'aws_agent'
    }

    environment {
        DOCKERHUB_CREDENTIALS = credentials('DuckerHub')
        DOCKER_IMAGE = "ahlamahmed/flask"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${DOCKER_IMAGE}:${env.BRANCH_NAME}")
                }
            }
        }
        stage('Push to Docker Hub') {
            steps {
                script {
                    docker.withRegistry('', 'DOCKERHUB_CREDENTIALS') {
                        docker.image("${DOCKER_IMAGE}:${env.BRANCH_NAME}").push()
                    }
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    def namespace
                    if (env.BRANCH_NAME == 'dev') {
                        namespace = 'dev'
                    } else if (env.BRANCH_NAME == 'test') {
                        namespace = 'test'
                    } else if (env.BRANCH_NAME == 'prod') {
                        namespace = 'prod'
                    } else {
                        error("Branch name not recognized: ${env.BRANCH_NAME}")
                    }

                    withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                        kubernetesDeploy(
                            configs: "k8s/${namespace}/deployment.yaml",
                            kubeConfig: [path: env.KUBECONFIG]
                        )
                        sh "kubectl set image deployment/your-deployment your-container=${DOCKER_IMAGE}:${env.BRANCH_NAME} --namespace=${namespace} --kubeconfig=${env.KUBECONFIG}"
                    }
                }
            }
        }
    }
}

