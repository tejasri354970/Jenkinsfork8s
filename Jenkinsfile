pipeline {
    agent any

    environment {
        GITHUB_REPO = 'https://github.com/your-username/your-repo.git'
        MAVEN_HOME = tool name: 'Maven', type: 'hudson.tasks.Maven$MavenInstallation'
        DOCKER_IMAGE_NAME = 'your-docker-image-name'
        DOCKER_HUB_CREDENTIALS = 'your-docker-hub-credentials-id'
        KUBE_CONFIG = credentials('your-kubeconfig-credentials-id')
    }

    stages {
        stage('Checkout') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/main']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: "${GITHUB_REPO}"]]])
            }
        }

        stage('Build') {
            steps {
                container('maven') {
                    sh "${MAVEN_HOME}/bin/mvn clean install"
                }
            }
        }

        stage('Docker Build & Push') {
            steps {
                script {
                    def dockerImage = docker.build("${DOCKER_IMAGE_NAME}:${env.BUILD_ID}")
                    docker.withRegistry("https://index.docker.io/v1/", DOCKER_HUB_CREDENTIALS) {
                        dockerImage.push()
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                container('kubectl') {
                    withCredentials([kubeconfigFile(credentialsId: "${KUBE_CONFIG}", variable: 'KUBECONFIG')]) {
                        sh "kubectl apply -f kubernetes-deployment.yaml"
                    }
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
