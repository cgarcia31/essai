pipeline {
     environment {
       ID_DOCKER = "${ID_DOCKER_PARAMS}"
       IMAGE_NAME = "essai_pipeline"
       IMAGE_TAG = "latest"
       STAGING = "${ID_DOCKER}-staging"
       PRODUCTION = "${ID_DOCKER}-production"
       DOCKER_IP = "192.168.118.134"
     }
     agent none
     stages {
         stage('Build image') {
             agent any
             steps {
                script {
		  echo "Build du container"
                  sh 'docker build -t ${DOCKER_ID}/${IMAGE_NAME}:${IMAGE_TAG} .'
                }
             }
        }
        stage('Run container based on builded image') {
            agent any
            steps {
               script {
                 sh '
		 	echo "Run du conteneur"
			docker rm -f ${IMAGE_NAME}
		 	docker run -d --name ${IMAGE_NAME} -p ${PORT_EXPOSED}:5000 -e PORT=5000 ${DOCKER_ID}/${IMAGE_NAME}:${IMAGE_TAG} '
               }
            }
       }
       stage('Test image') {
           agent any
           steps {
              script {
                sh 'curl -I http://${DOCKER_IP}:${PORT_EXPOSED} | grep 200 && curl -I http://${DOCKER_IP}:${PORT_EXPOSED} | grep -q "Hello world!"'
              }
           }
      }
      stage('Clean Container') {
          agent any
          steps {
             script {
               sh '''
	       	echo "Clean du container"
	       	docker stop ${IMAGE_NAME} && docker rm ${IMAGE_NAME}
             }
	     '''
          }
     }

     stage ('Login and Push Image on docker hub') {
          agent any
          environment {
	             DOCKERHUB_PASSWORD  = credentials('2bc257be-277d-4bc7-8873-02f774a04442')
          }       
          steps {
             script {
               sh '''
			echo $DOCKERHUB_PASSWORD_PSW | docker login -u $ID_DOCKER --password-stdin
			docker push ${ID_DOCKER}/$IMAGE_NAME:$IMAGE_TAG
               '''
             }
          }
      }    
     
     stage('Push image in staging and deploy it') {
       when {
              expression { GIT_BRANCH == 'origin/main' }
            }
      agent any
      environment {
          HEROKU_API_KEY = credentials('e445150d-679e-4af2-a11c-40e141cc60b9')
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
  }
}