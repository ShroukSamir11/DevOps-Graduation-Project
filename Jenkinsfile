pipeline {
  agent any
  options {
    buildDiscarder(logRotator(numToKeepStr: '5'))
  }
  environment {
    DOCKERHUB_CREDENTIALS = credentials('nexus')
  }
  stages {
    stage('Build') {
      steps {
          git branch: 'k8s_task', url: "https://github.com/mahmoud254/jenkins_nodejs_example.git"
          sh 'docker build -t k8s_task:latest .'
      }
    }
    // the ip is service ip "nexus-svd"
    stage('Login') {
      steps {
        sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login http://10.100.31.151:5000 -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
      }
    }
    stage('Push') {
      steps {
        sh "docker tag k8s_task:latest 10.100.31.151:5000/repository/grad-project/k8s_task:latest"
        sh 'docker push 10.100.31.151:5000/repository/grad-project/k8s_task:latest'
      }
    }
    stage('Creating Persistent Volume'){
        steps{
            sh 'kubectl apply -f k8s-yaml-files/volumes.yaml'
        }
    }
    stage('Deploying Database'){
        steps{
            sh 'kubectl apply -f k8s-yaml-files/secret.yaml'
            sh 'kubectl apply -f k8s-yaml-files/mysql-deployment.yaml'
        }
    }
    stage('Deploying Application'){
        steps{
            sh 'kubectl apply -f k8s-yaml-files/application.yaml'
            sh 'kubectl apply -f k8s-yaml-files/nodeport.yaml'
        }
    }
  }
  post {
    always {
      sh 'docker logout'
    }
  }
}
