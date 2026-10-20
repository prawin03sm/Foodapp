pipeline {
  agent any
  stages {
    stage('checkout') {
      steps {
        git "https://github.com/prawin03sm/Foodapp.git"
      }
    }
    stage('test') {
      steps {
        sh '''
            python3 -m venv venv
            . venv/bin/activate
            pip install -r requirements.txt
            python -m unittest discover -s tests
        '''
      }
    }
    stage('build') {
      steps {
        sh "docker build -t myimg ."
        sh "docker tag myimg prawin03sm/foodapp:1"
      }
    }
    stage('cont_test') {
      steps {
        sh '''#!/bin/bash
          docker run -itd --name test-cont -p 8081:5000 prawin03sm/foodapp:1
          sleep 3
          success=1
          for i in {1..10}; do
            echo "HTTP check attempt $i"
            if curl -f http://localhost:8081/; then
              success=0
              break
            fi
            sleep 2
          done
          if [ "$success" -ne 0 ]; then
            echo "HTTP Connection failed"
            exit 1
          fi
          echo "HTTP connection success"
        '''
      }
      post {
        always {
          sh "docker rm -f test-cont"
        }
      } 
    }
    stage('image_push') {
      steps {
        withCredentials([usernamePassword(
          credentialsId: 'dockerid',
          usernameVariable: 'dockerhub_user',
          passwordVariable: 'dockerhub_password'
        )]) {
          sh '''
            echo "$dockerhub_password" | docker login -u "$dockerhub_user" --password-stdin
            docker push prawin03sm/foodapp:1 
          '''
        }
      }
    }
    stage('deploy') {
      steps {
        sshagent(['jenkinsid']) {
          sh '''
            ssh ubuntu@10.10.1.48 '
              set -e
              docker pull prawin03sm/foodapp:1
              docker rm -f food || true
              docker run -d --name food -p 80:5000 prawin03sm/foodapp:1
	    '
          '''
        }
      }
    }
    stage('health check') {
      steps {
        sh '''#!/bin/bash
          success=0
          for i in {1..5}; do
            echo "Healthcheck attempt $i"
              if curl -f http://10.10.1.48:80/; then
                success=1
                break
              fi
              sleep 3
          done
          if [ "$success" -eq 1 ]; then
            echo "Application Healthy"
            exit 0
          fi
          echo "Application not Healthy"
          exit 1
        '''
      }
    }
  }
}
