pipeline {
  agent {
    kubernetes {
      yamlFile 'build-agent.yaml'
      defaultContainer 'maven'
      idleMinutes 1
    }

  }
  stages {
    stage('Checkout') {
        steps {
            git branch: 'main', url: 'https://github.com/billhoph/dso-demo'
            stash includes: '**/*', name: 'source'
        }
      }
    stage('Checkov') {
        steps {
            withCredentials([string(credentialsId: 'PC_USER', variable: 'pc_user'),string(credentialsId: 'PC_PASSWORD', variable: 'pc_password')]) {
                script {
                    docker.image('bridgecrew/checkov:latest').inside("--entrypoint=''") {
                      unstash 'source'
                      try {
                          sh 'checkov -d . --use-enforcement-rules -o cli -o junitxml --output-file-path console,results.xml --bc-api-key \$pc_user::\$pc_password --repo-id  billhoph/dso-demo --branch main'
                          junit skipPublishingChecks: true, testResults: 'results.xml'
                      } catch (err) {
                          junit skipPublishingChecks: true, testResults: 'results.xml'
                          throw err
                      }
                    }
                }
            }
        }
    }
    stage('Build') {
      parallel {
        stage('Compile') {
          steps {
            container(name: 'maven') {
              sh 'mvn compile'
            }

          }
        }

      }
    }

    stage('Test') {
      parallel {
        stage('Unit Tests') {
          steps {
            container(name: 'maven') {
              sh 'mvn test'
            }

          }
        }

      }
    }

    stage('Package') {
      parallel {
        stage('Create Jarfile') {
          steps {
            container(name: 'maven') {
              sh 'mvn package -DskipTests'
            }

          }
        }

      }
    }

    stage('Deploy to Dev') {
      steps {
        sh 'echo done'
      }
    }

  }
  environment {
    PC_USER = '81c2fe59-72a3-4e55-bb1d-9cff8ecc7bb7'
    PC_PASSWORD = 'TffwVyMCuNGC/byo68KCYQzUOW0='
  }
  options {
        preserveStashes()
        timestamps()
    }
}