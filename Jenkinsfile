def run(String cmd) {
  if (isUnix()) { sh cmd } else { bat cmd }
}

pipeline {
  agent any
  stages {
    stage('Checkout') {
      steps {
        git branch: 'main', url: 'https://github.com/sfaximaram5/MyAppReact'
      }
    }
    stage('Build images') {
      steps {
        script {
          run('docker build -t myapp-server ./Server')
          run('docker build -t myapp-client ./Client')
        }
      }
    }
    stage('Deploy') {
      steps {
        script {
          run('docker compose down')
          run('docker compose up -d --build')
        }
      }
    }
  }
}