pipeline {
  agent any

  environment {
    PROJECT_NAME = "CI/CD Pipeline"
    BUILD_NUMBER = "${env.BUILD_NUMBER}"
  }
  
  stages {
    
    stage('Cleanup Workspace') {
            steps {
                echo '🧹 Cleaning workspace before build...'
                deleteDir()
                echo '✅ Workspace cleaned'
            }
    }
    
    stage('git_checkout') {
      steps {
        checkout scm
        sh '''
           echo "Repo checked out successfully"
           echo "Current branch: $(git branch --show-current || echo 'detached HEAD')"
           echo "Commit: $(git rev-parse --short HEAD)"
           echo ""
           echo "Project structure:"
           ls -la
        '''
      }
    }
    
    stage('Application Build') {
      steps {
        sh '''
           set -e
           test -f scripts/build.sh
           
           echo "🚀 Building application..."
           chmod +x scripts/build.sh
           ./scripts/build.sh
          '''
        }
    } 

  }
}
