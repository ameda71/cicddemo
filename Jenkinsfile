pipeline {
    agent any

    environment {
        IMAGE_TAG = "v${BUILD_NUMBER}"
        DOCKER_HUB_REPO = "saiteja562/samplewebapp2"
    }

    stages {
        stage('Stage 1. Checkout source code repo') {
            steps {
                git branch: 'main',
                    credentialsId: 'github', // ✅ Your Global GitHub creds
                    url: 'git@github.com:ameda71/cicddemo.git'
            }
        }

        stage('Stage 2. Build Docker image') {
            steps {
                script {
                    sh '''
                    cd cicd/Django-webapp
                    echo 'Build Docker Image'
                    docker build -t ${DOCKER_HUB_REPO}:${IMAGE_TAG} .
                    '''
                }
            }
        }

        stage('Stage 3. Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub', usernameVariable: 'DOCKERHUB_USER', passwordVariable: 'DOCKERHUB_PASS')]) {
                    script {
                        sh '''
                        echo $DOCKERHUB_PASS | docker login -u $DOCKERHUB_USER --password-stdin
                        docker push ${DOCKER_HUB_REPO}:${IMAGE_TAG}
                        docker logout
                        '''
                    }
                }
            }
        }

        stage('Stage 4. Update K8s manifest and push to GitHub') {
            steps {
                script {
                    sh '''
                    cd cicd/ArgoCD/deployments/
                    echo "Before update:"
                    cat django-app-pod-svc.yaml

                    sed -i "s|replacetag|${IMAGE_TAG}|g" django-app-pod-svc.yaml

                    echo "After update:"
                    cat django-app-pod-svc.yaml

                    git config --global user.name "jenkins"
                    git config --global user.email "jenkins@example.com"

                    git add django-app-pod-svc.yaml
                    git commit -m "Update image tag to ${IMAGE_TAG} via Jenkins"
                    git push origin main
                    '''
                }
            }
        }
    }
}
