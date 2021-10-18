pipeline {
     environment {
       IMAGE_NAME = "alpinehelloworld"
       IMAGE_TAG = "latest"
       STAGING = "pintade-staging"
       PRODUCTION = "pintade-production"
	   docker_user = "pintade"
     }
     agent { label 'master' }
     stages {
         stage('Build image') {
             agent any
             steps {
                script {
                  sh 'docker build -t pintade/$IMAGE_NAME:$IMAGE_TAG .'
                }
             }
        }
        stage('Run container based on builded image') {
            agent any
            steps {
               script {
                 sh '''
                    docker run --name $IMAGE_NAME -d -p 80:5000 -e PORT=5000 pintade/$IMAGE_NAME:$IMAGE_TAG
                    sleep 5
                 '''
               }
            }
       }
       stage('Test image') {
           agent any
           steps {
              script {
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
                 docker stop $IMAGE_NAME
                 docker rm $IMAGE_NAME
				 docker push 
               '''
             }
          }
     }
     stage('Push image in staging and deploy it') {
       when {
              expression { GIT_BRANCH == 'origin/master' }
            }
      agent any
      environment {
          HEROKU_API_KEY = credentials('heroku_api_key')
      }  
      steps {
          script {
            sh '''
              heroku container:login
              heroku create $STAGING || echo "project already exist"
              heroku container:push -a $STAGING web
              heroku container:release -a $STAGING web
            '''
          }
        }
     }
     stage('Push image in production and deploy it') {
       when {
              expression { GIT_BRANCH == 'origin/master' }
            }
      agent any
      environment {
          HEROKU_API_KEY = credentials('heroku_api_key')
      }  
      steps {
          script {
            sh '''
              heroku container:login
              heroku create $PRODUCTION || echo "project already exist"
              heroku container:push -a $PRODUCTION web
              heroku container:release -a $PRODUCTION web
            '''
          }
        }
     }
	  stage('Push docker') {
          agent any
          steps {
             script {
               sh '''
				docker login -u ${docker_user} -p credentials('docker_pw')
				docker push 
               '''
             }
          }
     }
  }
}