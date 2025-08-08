pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "abelitang/springboot-openshift:latest"
        OC_HPA_PATH = "/home/odp/bintang/hpa.yml"
    }

    stages {
        stage('Build and Push') {
            steps {
                echo 'Build & Push'
                sh '''
                    docker build springboot-openshift:latest . &&\
                    docker tag ${DOCKER_IMAGE} &&\
                    docker push                 
                '''
            }
        }
        stage('OC HPA Delete & Push OC HPA'){
            steps{
                echo 'Deleting . . . & Pushing . . .'
                sh '''
                    oc delete -f ${OC_HPA_PATH} &&\
                    oc apply -f ${OC_HPA_PATH}
                '''
            }
        }
        stage('GET ROUTE'){
            steps{
                echo 'this is your route'
                sh '''
                    oc get route
                '''
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
