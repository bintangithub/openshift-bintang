pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = 'abelitang/springboot-openshift'
        DOCKER_TAG = 'latest'
        OC_PROJECT = 'spring-openshift'
        OC_HPA_PATH = './hpa.yml'
    }
    
    stages {
        stage('Build and Push Docker Image') {
            steps {
                script {
                    // Build Docker image
                    sh "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} ."
                    
                    // Login to Docker Hub
                    withCredentials([usernamePassword(
                        credentialsId: 'docker-hub-bintang', 
                        usernameVariable: 'DOCKER_USER', 
                        passwordVariable: 'DOCKER_PASS'
                    )]) {
                        sh "docker login -u ${DOCKER_USER} -p ${DOCKER_PASS}"
                    }
                    
                    // Push image
                    sh "docker push ${DOCKER_IMAGE}:${DOCKER_TAG}"
                }
            }
        }
        
        stage('OpenShift Deployment') {
            steps {
                script {
                    // Login to OpenShift
                    withCredentials([string(credentialsId: 'token-bintang', variable: 'TOKEN')]) {
                        sh "oc login --token=${TOKEN} --server=https://api.rm3.7wse.p1.openshiftapps.com:6443"
                    }
            
                    // Clean up existing resources
                    sh """
                        oc delete deployment springboot-openshift || true
                        oc delete service springboot-openshift || true
                        oc delete route springboot-openshift || true
                        oc delete imagestream springboot-openshift || true
                        oc delete bc springboot-openshift || true
                    """

                    // Deploy using new-app
                    sh "oc new-app ${DOCKER_IMAGE}:${DOCKER_TAG}"
                    
                    // Apply HPA
                    sh "oc delete hpa -f ${OC_HPA_PATH} || true"
                    sh "oc apply -f ${OC_HPA_PATH}"
                    
                    // Get route
                    sh "oc get route"
                }
            }
        }
    }
    
    post {
        always {
            // Cleanup
            sh "docker logout"
        }
    }
}