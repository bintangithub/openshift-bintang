pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "abelitang/springboot-openshift:latest"
        OC_HPA_PATH = "/home/odp/bintang/hpa.yml"
        DOCKER_REGISTRY = "docker.io"
        PROJECT_NAME = "spring-openshift"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: "main", url: "https://github.com/bintangithub/openshift-bintang.git"
            }
        }
        
        stage('Build and Push Docker Image') {
            steps {
                echo 'Building, Tagging & Pushing Docker Image'
                sh '''
                    docker build -t ${DOCKER_IMAGE} .
                    docker login ${DOCKER_REGISTRY} -u your_username -p your_password
                    docker push ${DOCKER_IMAGE}
                '''
            }
        }
        
        stage('Delete and Create OpenShift Project') {
            steps {
                echo 'Deleting and recreating OpenShift project'
                sh '''
                    oc delete project ${PROJECT_NAME} || true
                    oc new-project ${PROJECT_NAME}
                    oc new-app ${DOCKER_IMAGE}
                '''
            }
        }
        
        stage('Manage HPA') {
            steps {
                echo 'Deleting and applying HPA'
                sh '''
                    oc delete -f ${OC_HPA_PATH} || true
                    oc apply -f ${OC_HPA_PATH}
                '''
            }
        }
        
        stage('Get Route') {
            steps {
                echo 'This is your route'
                sh 'oc get route'
            }
        }
    }

    post {
        success {
            echo 'Deployment successful ✅'
        }
        failure {
            echo 'Deployment failed ❌'
        }
    }
}