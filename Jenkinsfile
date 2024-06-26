pipeline {
environment { // Declaration of environment variables
DOCKER_ID = "sebdevops06" // replace this with your docker-id
DOCKER_IMAGE0 = "movie-service"
DOCKER_IMAGE1 = "cast-service"
DOCKER_TAG = "v.${BUILD_ID}.0" // we will tag our images with the current build in order to increment the value by 1 with each new build
}
agent any // Jenkins will be able to select all available agents
stages {
        stage(' Docker Build movie-service'){ // docker build image stage
            steps {
                script {
                sh '''
                 cd movie-service
                 docker rm -f jenkins
                 docker build -t $DOCKER_ID/$DOCKER_IMAGE0:$DOCKER_TAG .
                sleep 6
                '''
                }
            }
        }
        stage('Docker run movie-service'){ // run container from our builded image
                steps {
                    script {
                    sh '''
                    cd movie-service
                    docker run -d -p 80:80 --name jenkins $DOCKER_ID/$DOCKER_IMAGE0:$DOCKER_TAG
                    sleep 10
                    '''
                    }
                }
            }

        stage('Test Acceptance movie-service'){ // we launch the curl command to validate that the container responds to the request
            steps {
                    script {
                    sh '''
                    curl localhost
                    '''
                    }
            }

        }
        stage('Docker Push movie-service'){ //we pass the built image to our docker hub account
            environment
            {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS") // we retrieve  docker password from secret text called docker_hub_pass saved on jenkins
            }

            steps {

                script {
                sh '''
                docker login -u $DOCKER_ID -p $DOCKER_PASS
                docker push $DOCKER_ID/$DOCKER_IMAGE0:$DOCKER_TAG
                '''
                }
            }

        }
        stage(' Docker Build cast-service'){ // docker build image stage
            steps {
                script {
                sh '''
                 cd cast-service
                 docker rm -f jenkins
                 docker build -t $DOCKER_ID/$DOCKER_IMAGE1:$DOCKER_TAG .
                sleep 6
                '''
                }
            }
        }
        stage('Docker run cast-service'){ // run container from our builded image
                steps {
                    script {
                    sh '''
                    cd cast-service
                    docker run -d -p 80:80 --name jenkins $DOCKER_ID/$DOCKER_IMAGE1:$DOCKER_TAG
                    sleep 10
                    '''
                    }
                }
            }

        stage('Test Acceptance cast-service'){ // we launch the curl command to validate that the container responds to the request
            steps {
                    script {
                    sh '''
                    curl localhost
                    '''
                    }
            }

        }
        stage('Docker Push cast-service'){ //we pass the built image to our docker hub account
            environment
            {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS") // we retrieve  docker password from secret text called docker_hub_pass saved on jenkins
            }

            steps {

                script {
                sh '''
                docker login -u $DOCKER_ID -p $DOCKER_PASS
                docker push $DOCKER_ID/$DOCKER_IMAGE1:$DOCKER_TAG
                '''
                }
            }

        }
stage('Deploiement en dev'){
        environment
        {
        KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
        }
            steps {
                script {
                sh '''
                rm -Rf .kube
                mkdir .kube
                ls
                cat $KUBECONFIG > .kube/config
                cp k3s/values.yml values.yml
                cat values.yml
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                helm upgrade --install app k3s --values=values.yml --namespace dev
                '''
                }
            }

        }
stage('Deploiement en staging'){
        environment
        {
        KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
        }
            steps {
               script {
                sh '''
                rm -Rf .kube
                mkdir .kube
                ls
                cat $KUBECONFIG > .kube/config
                cp k3s/values.yml values.yml
                cat values.yml
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                helm upgrade --install app k3s --values=values.yml --namespace staging
                '''
                }
            }

        }
  stage('Deploiement en qa'){
        environment
        {
        KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
        }
            steps {
                script {
                sh '''
                rm -Rf .kube
                mkdir .kube
                ls
                cat $KUBECONFIG > .kube/config
                cp k3s/values.yml values.yml
                cat values.yml
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                helm upgrade --install app k3s --values=values.yml --namespace qa
                '''
                }
            }

        }
  stage('Deploiement en prod'){
        environment
        {
        KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
        }
            steps {
            // Create an Approval Button with a timeout of 15minutes.
            // this require a manuel validation in order to deploy on production environment
                    timeout(time: 15, unit: "MINUTES") {
                        input message: 'Do you want to deploy in production ?', ok: 'Yes'
                    }

                script {
                sh '''
                rm -Rf .kube
                mkdir .kube
                ls
                cat $KUBECONFIG > .kube/config
                cp k3s/values.yml values.yml
                cat values.yml
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                helm upgrade --install app k3s --values=values.yml --namespace prod
                '''
                }
            }

        }
}
}