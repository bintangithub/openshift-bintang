pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = 'abelitang/springboot-openshift'
        DOCKER_TAG = 'latest'
        OC_PROJECT = 'spring-openshift'
        OC_HPA_PATH = './hpa.yaml'
    }
    
    stages {
        stage('Build and Push Docker Image') {
            steps {
                script {
                    // Build Docker image
                    sh "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} ."
                    
                    // Login to Docker Hub (gunakan credential Jenkins)
                    withCredentials([usernamePassword(credentialsId: 'docker-hub-bintang', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh "echo ${DOCKER_PASS} | docker login -u ${DOCKER_USER} --password-stdin"
                    }
                    
                    // Push image
                    sh "docker push ${DOCKER_IMAGE}:${DOCKER_TAG}"
                }
            }
        }
        
        stage('OpenShift Deployment') {
            steps {
                script {
                    sh "oc login --token=sha256~Ssm0xNRD-LQxsHh9GP5_gclKpatxV6_phmGN-9aGARg --server=https://api.rm3.7wse.p1.openshiftapps.com:6443"
                    // // Delete existing project if exists
                    // sh "oc delete project ${OC_PROJECT} || true"
                    
                    // // Create new project
                    // sh "oc new-project ${OC_PROJECT}"
                    
                    sh "oc delete all -l app=springboot-openshift"

                    // Deploy using new-app
                    sh "oc new-app ${DOCKER_IMAGE}:${DOCKER_TAG}"
                    
                    // Delete existing HPA if exists
                    sh "oc delete hpa -f ${OC_HPA_PATH} || true"
                    
                    // Apply new HPA
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