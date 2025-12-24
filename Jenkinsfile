pipeline {
  agent any
  stages {
    stage('git_checkout') {
      steps {
        checkout scm
        sh '''echo "Repo checked out"
        ls -la'''
      }
    }
    

  }
}
