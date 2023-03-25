pipeline {
  agent {
    kubernetes {
      yamlFile 'build-agent.yaml'
      defaultContainer 'maven'
      idleMinutes 1
    }

  }
  stages {
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

    stage('Static Analysis') {
      parallel {
        stage('Unit Tests') {
          steps {
            container(name: 'maven') {
              sh 'mvn test'
            }
          }
        }
        stage('SCA') {
          steps {
            container('maven') {
		          catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
               sh 'mvn org.owasp:dependency-check-maven:check'
              }
	          } 
	        }
          post {
            always {
              archiveArtifacts allowEmptyArchive: true, artifacts: 'target/dependency-check-report.html', fingerprint: true, onlyIfSuccessful: true
             // dependencyCheckPublisher pattern: 'report.xml'
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
        stage('Docker BnP') {
          steps {
            container('kaniko') {
              sh '/kaniko/executor -f `pwd`/Dockerfile -c `pwd` --insecure --skip-tls-verify --cache=true --destination=docker.io/joanjoho/dsodemo'
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
}