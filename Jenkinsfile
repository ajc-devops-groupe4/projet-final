pipeline {
  agent any
  environment {
    ID_DOCKER = "fredlab"
    IMAGE_NAME = "ic-webapp"
    PORT_EXPOSED = "80"
    ODOO = "${sh(script:'awk \'/ODOO/ {sub(/^.**URL/,\"\");print $2}\' releases.txt', returnStdout: true).trim()}"
    PGADMIN = "${sh(script:'awk \'/PGADMIN/ {sub(/^.**URL/,\"\");print $2}\' releases.txt', returnStdout: true).trim()}"
    VER = "${sh(script:'awk \'/version:/ {sub(/^.**version:/,\"\");print $1}\' releases.txt', returnStdout: true).trim()}"
  }
  stages {
    stage('Build ic-webapp image') {
      steps {
        script {
          sh '''
            docker build -f sources/docker/Dockerfile --build-arg odoo=${ODOO} --build-arg pgadmin=${PGADMIN} -t ${ID_DOCKER}/${IMAGE_NAME}:${VER} sources/docker             
            '''
        }
      }
    }
    stage('Run container based on builded image') {
      steps {
        script {
          sh '''
            echo "Clean Environment"
            docker rm -f $IMAGE_NAME || echo "container does not exist"
            docker run --name $IMAGE_NAME -d -p ${PORT_EXPOSED}:8080 ${ID_DOCKER}/$IMAGE_NAME:$VER
            sleep 5
             '''
        }
      }
    }
    stage('Test image') {
      steps {
        script {
          sh '''
            curl ${PGADMIN}:${PORT_EXPOSED} | grep -q "les titres de la page"
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
             '''
        }
      }
    }
    stage ('Login and Push Image on docker hub') {
      agent any
      environment {
        DOCKERHUB_PASSWORD  = credentials('dockerhub')
      }            
      steps {
        script {
          sh '''
            echo $DOCKERHUB_PASSWORD_PSW | docker login -u $ID_DOCKER --password-stdin
            docker push ${ID_DOCKER}/$IMAGE_NAME:$VER
             '''
        }
      }
    }    
    stage('Push images in staging and deploy it') {
      when {
        expression { GIT_BRANCH == 'origin/staging' }
      }
      steps {
        script {
          sh '''
            ansible-playbook sources/ansible/playbook_odoo.yml -i dev.yml
            ansible-playbook sources/ansible/playbook_pgadmin.yml -i dev.yml
            ansible-playbook sources/ansible/playbook_ic_webapp.yml -i dev.yml 
            '''
            // ansible-playbook playbook_ic_webapp.yml -i dev.yml --vault-password-file vault.key 
        }
      }
    }
    stage('Push images in production and deploy it') {
      when {
        expression { GIT_BRANCH == 'origin/master' }
      }
      steps {
        script {
          sh '''
            ansible-playbook sources/ansible/playbook_odoo.yml -i prod.yml
            ansible-playbook sources/ansible/playbook_pgadmin.yml -i prod.yml
            ansible-playbook sources/ansible/playbook_ic_webapp.yml -i prod.yml 
            '''
        }
      }
    }
  }
//   post {
//     success {
//       slackSend (
//         botUser: true,
//         color: '#00FF00', 
//         message: "SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})", 
//         tokenCredentialId: 'slack-token', 
//         channel: 'jenkins'
//       )
//     }
//     failure {
//       slackSend (
//         botUser: true,
//         color: '#FF0000', 
//         message: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})",
//         tokenCredentialId: 'slack-token', 
//         channel: 'jenkins'
//       )
//     }   
//   }
}