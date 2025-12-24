pipeline {
  agent any
  
  environment {
    JENKINS_UID = sh(script: "id -u jenkins", returnStdout: true).trim()
    JENKINS_GID = sh(script: "id -g jenkins", returnStdout: true).trim()
    PROJECT_NAME = "CI/CD Pipeline Demo"
    BUILD_NUMBER = "${env.BUILD_NUMBER}"
    IMAGE_NAME = "dauletshaimerdenov/node-demo"
    IMAGE_TAG  = "${BUILD_NUMBER}"
    DOCKER_AGENT_IMAGE = "docker:20.10.24" // образ с Docker CLI
    NPM_CONFIG_CACHE = "${WORKSPACE}/.npm"
    DOCKER_HUB_CREDENTIALS = 'dockerhub'
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
    script {
      echo "🚀 Building application with Node 7.8.0..."
      sh 'chmod +x scripts/build.sh'
      withEnv(["NPM_CONFIG_CACHE=${WORKSPACE}/.npm"]) {
        sh '''
          set -e
          test -f scripts/build.sh
          ./scripts/build.sh
        '''
      }
    }
  }
}
stage('Fix Permissions after Build') {
  agent {
    docker {
      image 'node:18'
      args '-u root:root'
    }
  }
  steps {
    sh '''
      if [ -d "node_modules" ]; then
        chown -R ${JENKINS_UID}:${JENKINS_GID} node_modules
      else
        echo "⚠️ node_modules not found, skipping chown"
      fi
    '''
  }
}
    
stage('Application Test') {
  agent {
    docker {
      image 'node:18'
      args "-u ${env.JENKINS_UID}:${env.JENKINS_GID} -v /var/run/docker.sock:/var/run/docker.sock"
    }
  }
  environment {
    NPM_CONFIG_CACHE = "${WORKSPACE}/.npm"
  }
  steps {
    sh '''
      echo "🚀 Installing dependencies..."
      npm install
      echo "🚀 Running tests..."
      npm test
    '''
  }
}

         stage('Build Docker Image') {
            agent {
                docker {
                    image "${DOCKER_AGENT_IMAGE}"
                    args "-v /var/run/docker.sock:/var/run/docker.sock"
                }
            }
            steps {
                script {
                    echo "📦 Building Docker image..."
                    sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
                }
            }
        }

            stage('Push Docker Image') {
            agent {
                docker {
                    image "${DOCKER_AGENT_IMAGE}"
                    args "-v /var/run/docker.sock:/var/run/docker.sock"
                }
            }
            steps {
                script {
                    echo "🔑 Logging in to Docker Hub..."
                    withCredentials([usernamePassword(credentialsId: "${DOCKER_HUB_CREDENTIALS}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                    }

                    echo "🚀 Pushing Docker image..."
                    sh "docker push ${IMAGE_NAME}:${IMAGE_TAG}"

                    // Тег latest
                    sh "docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${IMAGE_NAME}:latest"
                    sh "docker push ${IMAGE_NAME}:latest"
                }
            }
    
  }
}
}
