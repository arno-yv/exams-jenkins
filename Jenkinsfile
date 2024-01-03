pipeline {
environment {
DOCKER_ID = "arnoyv" 
DOCKER_CAST_IMAGE = "cast-api"
DOCKER_MOVIE_IMAGE = "movie-api"
DOCKER_TAG = "v.${BUILD_ID}.0" // we will tag our images with the current build in order to increment the value by 1 with each new build
}
agent any 
stages {
        stage(' Docker Build'){ // docker build images stage
            steps {
                script {
                sh '''
                 docker rm -f jenkins
                 cd movie
                 docker build -t $DOCKER_ID/$DOCKER_MOVIE_IMAGE:$DOCKER_TAG .
                 cd ../cast
                 docker build -t $DOCKER_ID/$DOCKER_CAST_IMAGE:$DOCKER_TAG .
                 sleep 6
                '''
                }
            }
        }
        stage('Docker run'){ // run containers from our builded images
                steps {
                    script {
                    sh '''
                    docker run -d -p 8001:8000 --name api_movie $DOCKER_ID/$DOCKER_MOVIE_IMAGE:$DOCKER_TAG
                    docker run -d -p 8002:8000 --name api_cast $DOCKER_ID/$DOCKER_CAST_IMAGE:$DOCKER_TAG
                    sleep 10
                    '''
                    }
                }
            }

        stage('Test Acceptance'){ // we launch the curl command to validate that containers respond to the request
            steps {
                    script {
                    sh '''
                    curl localhost:8001
                    curl localhost:8002

                    '''
                    }
            }

        }
        stage('Docker Push'){ //we pass the built images to our docker hub account
            environment
            {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS") // we retrieve  docker password from secret text called docker_hub_pass saved on jenkins
            }

            steps {

                script {
                sh '''
                docker login -u $DOCKER_ID -p $DOCKER_PASS
                docker push $DOCKER_ID/$DOCKER_MOVIE_IMAGE:$DOCKER_TAG
                docker push $DOCKER_ID/$DOCKER_CAST_IMAGE:$DOCKER_TAG

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
                cd movie
                cat values_dev.yml
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values_dev.yml
                helm upgrade --install app movie --values=values_dev.yml --namespace dev
                cd ../cast
                cat values_dev.yml
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values_dev.yml
                helm upgrade --install app cast --values=values_dev.yml --namespace dev
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
                cd movie
                cat values_qa.yml
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values_qa.yml
                helm upgrade --install app movie --values=values_qa.yml --namespace qa
                cd ../cast
                cat values_qa.yml
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values_qa.yml
                helm upgrade --install app cast --values=values_qa.yml --namespace qa
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
                cd movie
                cat values_staging.yml
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values_staging.yml
                helm upgrade --install app movie --values=values_staging.yml --namespace staging
                cd ../cast
                cat values_staging.yml
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values_staging.yml
                helm upgrade --install app cast --values=values_staging.yml --namespace staging
                '''
                }
            }

        }
        stage('Deploiement en prod'){
            environment
            {
            KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
            }
            // this require a manuel validation in order to deploy on production environment
            input{
                message 'Do you want to deploy in production ?'
                ok 'yes'
            }
            // deploy only is the branch is master
            when {
                branch 'master'
            }
            steps {
                script {
                sh '''
                rm -Rf .kube
                mkdir .kube
                ls
                cat $KUBECONFIG > .kube/config
                cd movie
                cat values_prod.yml
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values_prod.yml
                helm upgrade --install app movie --values=values_prod.yml --namespace prod
                cd ../cast
                cat values_prod.yml
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values_prod.yml
                helm upgrade --install app cast --values=values_prod.yml --namespace prod
                '''
                }
            }

        }

}
}