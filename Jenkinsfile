pipeline {
  agent any
  
  environment {
    PROJECT_NAME = "CI/CD Pipeline Demo"
    BUILD_NUMBER = "${env.BUILD_NUMBER}"
    IMAGE_NAME = "dauletshaimerdenov/node-demo"
    IMAGE_TAG  = "${BUILD_NUMBER}"]
    DOCKER_AGENT_IMAGE = "docker:20.10.24" // образ с Docker CLI
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
      agent {
        docker {
            image 'node:7.8.0'
            args "-u ${env.JENKINS_UID}:${env.JENKINS_GID} -v /var/run/docker.sock:/var/run/docker.sock"
        }
      }
            
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

    stage('Application Test') {
      agent {
        docker {
            image 'node:7.8.0'
            args "-u ${env.JENKINS_UID}:${env.JENKINS_GID} -v /var/run/docker.sock:/var/run/docker.sock"
        }
      }
      steps {
        sh '''
            chmod +x scripts/test.sh
            ./scripts/test.sh
        '''
      }
    }
    
  }
}
