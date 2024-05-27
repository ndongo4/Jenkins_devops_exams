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
                        docker build --progress=plain -t  nginx:latest  ./nginx-chart
                        sleep 6
                    '''
                }
            }
        }
        stage('Docker Run') {
            steps {
                script {
                    sh '''
                        docker run -d -p 8000:8000 --name jenkins-cast  $DOCKER_ID/cast-service:$DOCKER_TAG
                        docker run -d -p 8001:8000 --name jenkins-movie $DOCKER_ID/movie-service:$DOCKER_TAG
                        docker run -d -p 8080:8080 --name jenkins-nginx nginx:latest
                        sleep 10
                    '''
                }
            }
        }
        stage('Test Acceptance') {
            steps {
                script {
                    sh '''
                        curl localhost || true
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
                        helm upgrade --install nginx-dev nginx-chart --set image.tag=latest --namespace dev
                      '''
                }
            }
        }
        stage('Deploy to QA') {
       
            steps {
                script {
                    sh '''
                        export KUBECONFIG=$KUBECONFIG_FILE
                        helm upgrade --install cast-service-qa cast-service-chart --set image.tag=$DOCKER_TAG --namespace qa
                        helm upgrade --install movie-service-qa movie-service-chart --set image.tag=$DOCKER_TAG --namespace qa
                        helm upgrade --install nginx-qa nginx-chart --set image.tag=latest --namespace qa
                     '''
                }
            }
        }
        stage('Deploy to Staging') {
           
            steps {
                script {
                    sh '''
                        export KUBECONFIG=$KUBECONFIG_FILE
                        helm upgrade --install cast-service-staging cast-service-chart --set image.tag=$DOCKER_TAG --namespace staging
                        helm upgrade --install movie-service-staging movie-service-chart --set image.tag=$DOCKER_TAG --namespace staging
                        helm upgrade --install nginx-staging nginx-chart --set image.tag=latest --namespace staging 
                    '''
                }
            }
        }
        stage('Deploy to Prod') {
             // when {
             //   branch 'master'
             //}
            steps {
                timeout(time: 15, unit: 'MINUTES') {
                    input message: 'Do you want to deploy to production?', ok: 'Yes'
                }
                script {
                    sh '''
                        export KUBECONFIG=$KUBECONFIG_FILE
                        helm upgrade --install cast-service-prod cast-service-chart --set image.tag=$DOCKER_TAG --namespace prod
                        helm upgrade --install movie-service-prod movie-service-chart --set image.tag=$DOCKER_TAG --namespace prod
                        helm upgrade --install nginx-prod nginx-chart --set image.tag=latest --namespace prod
                     '''
                }
            }
        }
    }
    post {
        

        failure {
            echo "This will run if the job failed"
            mail to: "seye.ndongo4@gmail.com",
                 subject: "${env.JOB_NAME} - Build # ${env.BUILD_ID} has failed",
                 body: "For more info on the pipeline failure, check out the console output at ${env.BUILD_URL}"
        }
    }
}
