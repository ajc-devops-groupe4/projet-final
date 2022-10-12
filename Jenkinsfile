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
            docker build --no-cache -f sources/docker/Dockerfile --build-arg odoo="${ODOO}:8069" --build-arg pgadmin="${PGADMIN}:8070" -t ${ID_DOCKER}/${IMAGE_NAME}:${VER} sources/docker             
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
      environment {
        PRIVATE_KEY = credentials('private-key-ansible')
        VAULT_KEY = credentials('vault-key')
      }
      steps {
        script {
          sh '''
            rm -f vault-key id_rsa
            cp $PRIVATE_KEY id_rsa
            cp $VAULT_KEY vault-key
            chmod 400 id_rsa vault-key
            ansible-playbook sources/ansible/playbook_odoo.yml -i sources/ansible/dev.yml -e version=$VER --vault-password-file vault-key
            ansible-playbook sources/ansible/playbook_pgadmin.yml -i sources/ansible/dev.yml -e version=$VER --vault-password-file vault-key
            ansible-playbook sources/ansible/playbook_ic_webapp.yml -i sources/ansible/dev.yml -e version=$VER --vault-password-file vault-key
            '''
          timeout(time: 60, unit: "MINUTES") {
            input message: "L'appli est-elle fonctionnelle ?", ok: 'Yes'
          } 
        }
      }
    }
    stage('Push images in production and deploy it') {
      when {
        expression { GIT_BRANCH == 'origin/master' }
      }
      environment {
        PRIVATE_KEY = credentials('private-key-ansible')
        VAULT_KEY = credentials('vault-key')
      }
      steps {
        script {
          sh '''
            rm -f vault-key id_rsa
            cp $PRIVATE_KEY id_rsa
            cp $VAULT_KEY vault-key
            chmod 400 id_rsa vault-key
            ansible-playbook sources/ansible/playbook_odoo.yml -i sources/ansible/prod.yml -e version=$VER --vault-password-file vault-key
            ansible-playbook sources/ansible/playbook_pgadmin.yml -i sources/ansible/prod.yml -e version=$VER --vault-password-file vault-key
            ansible-playbook sources/ansible/playbook_ic_webapp.yml -i sources/ansible/prod.yml -e version=$VER --vault-password-file vault-key
            '''
        }
      }
    }
  }
  post {
    success {
      slackSend (
        botUser: true,
        color: '#00FF00', 
        message: "SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})", 
        tokenCredentialId: 'slack-jenkins', 
        channel: 'jenkins'
      )
    }
    failure {
      slackSend (
        botUser: true,
        color: '#FF0000', 
        message: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})",
        tokenCredentialId: 'slack-jenkins', 
        channel: 'jenkins'
      )
    }   
  }
}