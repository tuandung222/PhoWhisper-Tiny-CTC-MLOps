pipeline {
    agent any

    parameters {
        string(name: 'VERSION', defaultValue: '', description: 'Image version tag (default: timestamp)')
    }

    environment {
        DOCKER_HUB_CREDS = credentials('docker-hub-credentials')
        DO_API_TOKEN = credentials('do-api-token')
        CLUSTER_NAME = 'asr-k8s-cluster'
        DOCKER_REGISTRY = 'tuandung12092002'
        API_IMAGE = 'asr-fastapi-server'
        UI_IMAGE = 'asr-streamlit-ui'
        LATEST_TAG = 'latest'
    }

    stages {
        stage('Init') {
            steps {
                script {
                    if (params.VERSION) {
                        env.VERSION = params.VERSION
                    } else {
                        env.VERSION = sh(script: 'date +"%s"', returnStdout: true).trim()
                    }
                }
            }
        }
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build and Push Images') {
            steps {
                sh 'chmod +x ./push_images.sh'
                sh 'echo $DOCKER_HUB_CREDS_PSW | docker login -u $DOCKER_HUB_CREDS_USR --password-stdin'

                // Build and push API image
                sh '''
                    docker build \
                        --build-arg APP_USER=api \
                        --build-arg APP_USER_UID=1000 \
                        -t $DOCKER_REGISTRY/$API_IMAGE:$VERSION \
                        -t $DOCKER_REGISTRY/$API_IMAGE:$LATEST_TAG \
                        -f api/Dockerfile .
                    docker push $DOCKER_REGISTRY/$API_IMAGE:$VERSION
                    docker push $DOCKER_REGISTRY/$API_IMAGE:$LATEST_TAG
                '''

                // Build and push UI image
                sh '''
                    docker build \
                        --build-arg APP_USER=streamlit \
                        --build-arg APP_USER_UID=1000 \
                        -t $DOCKER_REGISTRY/$UI_IMAGE:$VERSION \
                        -t $DOCKER_REGISTRY/$UI_IMAGE:$LATEST_TAG \
                        -f ui/Dockerfile .
                    docker push $DOCKER_REGISTRY/$UI_IMAGE:$VERSION
                    docker push $DOCKER_REGISTRY/$UI_IMAGE:$LATEST_TAG
                '''
            }
        }

        // stage('Deploy to Kubernetes') {
        //     steps {
        //         sh 'doctl auth init -t $DO_API_TOKEN'
        //         sh 'doctl kubernetes cluster kubeconfig save $CLUSTER_NAME'
        //         sh 'kubectl apply -f k8s/monitoring/observability-namespace.yaml'
        //         sh 'kubectl apply -f k8s/base/namespace.yaml'
        //         sh 'kubectl apply -f k8s/base/'
        //         sh 'kubectl set image deployment/asr-api asr-api=$DOCKER_REGISTRY/$API_IMAGE:$VERSION -n asr-system'
        //         sh 'kubectl set image deployment/asr-ui asr-ui=$DOCKER_REGISTRY/$UI_IMAGE:$VERSION -n asr-system'
        //         sh 'kubectl rollout restart deployment/asr-api -n asr-system'
        //         sh 'kubectl rollout restart deployment/asr-ui -n asr-system'
        //         sh 'kubectl rollout status deployment/asr-api -n asr-system'
        //         sh 'kubectl rollout status deployment/asr-ui -n asr-system'
        //     }
        // }

        // stage('Deploy Monitoring') {
        //     steps {
        //         sh '''
        //             helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
        //             helm repo update
        //             helm upgrade --install prometheus prometheus-community/kube-prometheus-stack \
        //                 --namespace monitoring --create-namespace \
        //                 --values k8s/monitoring/prometheus-values.yaml
        //         '''
        //         sh '''
        //             helm repo add jaegertracing https://jaegertracing.github.io/helm-charts
        //             helm repo update
        //             helm upgrade --install jaeger-operator jaegertracing/jaeger-operator \
        //                 --namespace observability --create-namespace
        //             kubectl apply -f k8s/monitoring/jaeger-instance.yaml
        //         '''
        //     }
        // }

        // stage('Verify Deployment') {
        //     steps {
        //         sh 'kubectl get services -n asr-system'
        //         sh 'kubectl get services -n monitoring'
        //         sh 'kubectl get services -n observability'
        //         sh 'kubectl get pods -n asr-system'
        //         sh 'kubectl get pods -n monitoring'
        //         sh 'kubectl get pods -n observability'
        //     }
        // }
    }

    post {
        always {
            sh 'docker logout'
        }
        success {
            echo 'Deployment completed successfully!'
        }
        failure {
            echo 'Deployment failed!'
        }
    }
}
