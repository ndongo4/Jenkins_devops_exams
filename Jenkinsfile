pipeline {
    environment {
        DOCKER_ID = "nseye"
        DOCKER_TAG = "v.${BUILD_ID}.0"
        DOCKERHUB_CREDENTIALS = credentials('DOCKER_HUB_PASS')
        KUBECONFIG_FILE = credentials('config')
    }
    agent any
    stages {
        stage('Cleanup Before Build') {
            steps {
                script {
                    sh '''
                        docker container rm -f $(docker ps -a -q) || true
                        docker system prune -a -f --volumes || true
                    '''
                }
            }
        }
        stage('Docker Build') {
            steps {
                script {
                    sh '''
                        docker build -t $DOCKER_ID/cast-service:$DOCKER_TAG ./cast-service
                        docker build -t $DOCKER_ID/movie-service:$DOCKER_TAG ./movie-service
                        sleep 6
                    '''
                }
            }
        }
        stage('Docker Run') {
            steps {
                script {
                    sh '''
                        docker run -d -p 80:80 --name jenkins-cast $DOCKER_ID/cast-service:$DOCKER_TAG
                        docker run -d -p 80:80 --name jenkins-movie $DOCKER_ID/movie-service:$DOCKER_TAG
                        sleep 10
                    '''
                }
            }
        }
        stage('Test Acceptance') {
            steps {
                script {
                    sh '''
                        curl localhost
                    '''
                }
            }
        }
        stage('Docker Push') {
            steps {
                script {
                    sh '''
                        docker login -u $DOCKER_ID -p $DOCKERHUB_CREDENTIALS
                        docker push $DOCKER_ID/cast-service:$DOCKER_TAG
                        docker push $DOCKER_ID/movie-service:$DOCKER_TAG
                    '''
                }
            }
        }
        stage('Deploy to Dev') {
            
            steps {
                script {
                    sh '''
                        export KUBECONFIG=$KUBECONFIG_FILE
                        helm upgrade --install cast-service-dev cast-service-chart --set image.tag=$DOCKER_TAG --namespace dev
                        helm upgrade --install movie-service-dev movie-service-chart --set image.tag=$DOCKER_TAG --namespace dev
                    '''
                }
            }
        }
        stage('Deploy to QA') {
            environment {
                KUBECONFIG = KUBECONFIG_FILE
            }
            steps {
                script {
                    sh '''
                        export KUBECONFIG=$KUBECONFIG_FILE
                        helm upgrade --install cast-service-qa cast-service-chart --set image.tag=$DOCKER_TAG --namespace qa
                        helm upgrade --install movie-service-qa movie-service-chart --set image.tag=$DOCKER_TAG --namespace qa
                    '''
                }
            }
        }
        stage('Deploy to Staging') {
            environment {
                KUBECONFIG = KUBECONFIG_FILE
            }
            steps {
                script {
                    sh '''
                        export KUBECONFIG=$KUBECONFIG_FILE
                        helm upgrade --install cast-service-staging cast-service-chart --set image.tag=$DOCKER_TAG --namespace staging
                        helm upgrade --install movie-service-staging movie-service-chart --set image.tag=$DOCKER_TAG --namespace staging
                    '''
                }
            }
        }
        stage('Deploy to Prod') {
            when {
                branch 'master'
            }
            environment {
                KUBECONFIG = KUBECONFIG_FILE
            }
            steps {
                timeout(time: 15, unit: 'MINUTES') {
                    input message: 'Do you want to deploy to production?', ok: 'Yes'
                }
                script {
                    sh '''
                        export KUBECONFIG
                        helm upgrade --install cast-service-prod cast-service-chart --set image.tag=$DOCKER_TAG --namespace prod
                        helm upgrade --install movie-service-prod movie-service-chart --set image.tag=$DOCKER_TAG --namespace prod
                    '''
                }
            }
        }
    }
    post {
        always {
            cleanWs()
        }
        failure {
            echo "This will run if the job failed"
            mail to: "seye.ndongo4@gmail.com",
                 subject: "${env.JOB_NAME} - Build # ${env.BUILD_ID} has failed",
                 body: "For more info on the pipeline failure, check out the console output at ${env.BUILD_URL}"
        }
    }
}
