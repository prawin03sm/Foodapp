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
        sh "docker tag myimg prawin03sm/myimg:1"
      }
    }
    stage('cont_test') {
      steps {
        sh '''
          docker run -itd --name test-cont -p 8081:5000 myimg:1
          success=1
          for i in {1..10}; do
            echo "HTTP check attempt $i"
            if curl -f http://localhost:8081/; then
              success=0
              break
            fi
            sleep 2
          done
          if ["$success" -ne 0]; then
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
            docker push prawin03sm/myimg:1 
          '''
        }
      }
    }
  }
}
