pipeline {
    agent {
        label 'docker'
    }
    environment {
        DOCKERHUB_CREDS = credentials('dockerhub-creds')
        DOCKERHUB_USER = "${DOCKERHUB_CREDS_USR}"
        DOCKER_USER = "${DOCKERHUB_CREDS_USR}"
        DOCKER_PASS = "${DOCKERHUB_CREDS_PSW}"
        REACT_IMAGE = "${DOCKERHUB_USER}/ui"
        API_IMAGE = "${DOCKERHUB_USER}/api"

        JFROG_USER = 'anuharish52@gmail.com'
        JFROG_PASSWORD = credentials('jfrog-api-token') 
        HELM_REPO_URL = 'https://trialssd665.jfrog.io/artifactory/api-ui-helm-local'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/kevalohith/python-reach-sample.git'
            }
        }

        stage('Build Docker Images') {
            steps {
                sh '''
                    echo "Building React UI image..."
                    docker build -t ${REACT_IMAGE}:${BUILD_NUMBER} ./react-ui

                    echo "Building Flask API image..."
                    docker build -t ${API_IMAGE}:${BUILD_NUMBER} ./api-server-flask
                '''
            }
        }

        stage('Push Docker Images') {
            steps {
                sh '''
                    echo "Pushing Docker images..."
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    docker push ${REACT_IMAGE}:${BUILD_NUMBER}
                    docker push ${API_IMAGE}:${BUILD_NUMBER}
                '''
            }
        }

        stage('Deploy via Helm') {
            agent {
                label 'K8Master'
            }
            steps {
                sh '''
                    sudo su -
                    echo "Upgrading Helm release with new image tags..."
                    helm upgrade api-ui ./api-ui \
                      --install \
                      --namespace default \
                      --set ui.image.tag=${BUILD_NUMBER} \
                      --set api.image.tag=${BUILD_NUMBER}
                '''
            }
        }

        stage('Package & Push Helm Charts') {
            agent {
                label 'K8Master'
            }
            steps {
                sh '''
                    sudo su -
                    echo "Packaging Helm chart..."
                    helm package api-ui -d .

                    echo "Pushing Helm chart to JFrog..."
                    curl -u "$JFROG_USER:$JFROG_PASSWORD" -T api-ui-*.tgz ${HELM_REPO_URL}/api-ui-helm-local${BUILD_NUMBER}.tgz
                '''
            }
        }
    }
}
