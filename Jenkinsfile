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
                 cd movie/movie-service
                 docker build -t $DOCKER_ID/$DOCKER_MOVIE_IMAGE:$DOCKER_TAG .
                 cd ../../cast/cast-service
                 docker build -t $DOCKER_ID/$DOCKER_CAST_IMAGE:$DOCKER_TAG .
                 sleep 6
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
                cd
                rm -Rf .kube
                mkdir .kube
                ls
                cat $KUBECONFIG > .kube/config
                cd /home/ubuntu/examen/movie/movie-api
                sudo sed -i -e "s+tag.*+tag: ${DOCKER_TAG}+g" values.yaml
                cd ..
                helm upgrade --install ${BUILD_ID} movie-api --values=movie-api/values.yaml --namespace dev
                cd /home/ubuntu/examen/cast/cast-api
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yaml
                cd ..
                helm upgrade --install ${BUILD_ID} cast-api --values=cast-api/values.yaml --namespace dev
                cd /home/ubuntu/examen/nginx/
                helm upgrade --install ${BUILD_ID} nginx-api --values=nginx-api/values_dev.yaml --namespace dev

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
                cd
                rm -Rf .kube
                mkdir .kube
                ls
                cat $KUBECONFIG > .kube/config
                cd /home/ubuntu/examen/movie/movie-api
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yaml
                cd ..
                helm upgrade --install ${BUILD_ID} movie-api --values=movie-api/values.yaml --namespace qa
                cd /home/ubuntu/examen/cast/cast-api
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yaml
                cd ..
                helm upgrade --install ${BUILD_ID} cast-api --values=cast-api/values.yaml --namespace qa
                cd /home/ubuntu/examen/nginx/
                helm upgrade --install ${BUILD_ID} nginx-api --values=nginx-api/values_qa.yaml --namespace qa

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
                cd
                rm -Rf .kube
                mkdir .kube
                ls
                cat $KUBECONFIG > .kube/config
                cd /home/ubuntu/examen/movie/movie-api
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yaml
                cd ..
                helm upgrade --install ${BUILD_ID} movie-api --values=movie-api/values.yaml --namespace staging
                cd /home/ubuntu/examen/cast/cast-api
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yaml
                cd ..
                helm upgrade --install ${BUILD_ID} cast-api --values=cast-api/values.yaml --namespace staging
                cd /home/ubuntu/examen/nginx/
                helm upgrade --install ${BUILD_ID} nginx-api --values=nginx-api/values_staging.yaml --namespace staging

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
                cd
                rm -Rf .kube
                mkdir .kube
                ls
                cat $KUBECONFIG > .kube/config
                cd /home/ubuntu/examen/movie/movie-api
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yaml
                cd ..
                helm upgrade --install ${BUILD_ID} movie-api --values=movie-api/values.yaml --namespace prod
                cd /home/ubuntu/examen/cast/cast-api
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yaml
                cd ..
                helm upgrade --install ${BUILD_ID} cast-api --values=cast-api/values.yaml --namespace prod
                cd /home/ubuntu/examen/nginx/
                helm upgrade --install ${BUILD_ID} nginx-api --values=nginx-api/values_prod.yaml --namespace prod

                '''
                }
            }

        }

}
}