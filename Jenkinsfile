pipeline{

    environment{
        IMAGE_NAME = "sadofrazer/alpinehelloworld"
        IMAGE_TAG = "${BUILD_TAG}"
        CONTAINER_NAME = "alpinehelloworld"
        STAGING = "ajc-staging"
        PRODUCTION = "ajc-production"

    }

    agent none

    stages{

        stage ('Build Imahe'){
            agent any
            steps{
                script{
                    sh 'docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .'
                }
            }
        }

        stage ('Run container based on Builded image'){
            agent any
            steps{
                script{
                    sh '''
                        docker run --name ${CONTAINER_NAME} -d -p 80:5000 -e PORT=5000 ${IMAGE_NAME}:${IMAGE_TAG}
                        sleep 5
                    ''' 
                }
            }
        }

        stage ('Test Image'){
            agent any
            steps{
                script{
                    sh '''
                       curl http://172.17.0.1 | grep -q "Hello world!"
                    '''
                }
            }
        }

        stage('Clean Container') {
            agent any
            steps {
                script {
                    sh '''
                       docker stop ${CONTAINER_NAME}
                       docker rm ${CONTAINER_NAME}
                    '''
                }
            }
        }

        stage('Push image in staging and deploy it'){
            when {
                expression { GIT_BRANCH == 'origin/master'}
            }
            agent any
            environment{
                HEROKU_API_KEY = credentials('heroku_api_key')
            }
            steps{
                script{
                    sh '''
                       heroku container:login
                       heroku create ${STAGING} || echo "project already exist"
                       heroku container:push -a ${STAGING} web
                       heroku container:release -a ${STAGING} web
                    '''
                }
            }
        }

        stage('Push image in production and deploy it'){
            when {
                expression { GIT_BRANCH == 'origin/master'}
            }
            agent any
            environment{
                HEROKU_API_KEY = credentials('heroku_api_key')
            }
            steps{
                script{
                    sh '''
                       heroku container:login
                       heroku create ${PRODUCTION} || echo "project already exist"
                       heroku container:push -a ${PRODUCTION} web
                       heroku container:release -a ${PRODUCTION} web
                    '''
                }
            }
        }
        
    }

}