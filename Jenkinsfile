pipeline {
  agent {
    kubernetes {
      yamlFile 'build-agent.yaml'
      defaultContainer 'maven'
      idleMinutes 1
    }

  }
  stages {
    stage('Generate SBOM') {
      steps {
        container('maven') {
          sh 'mvn org.cyclonedx:cyclonedx-maven-plugin:makeAggregateBom'
        }
      }
      post {
        success {
          dependencyTrackPublisher projectName: 'sample-spring-app', 
          projectVersion: '0.0.1', 
          artifact: 'target/bom.xml', 
          autoCreateProjects: true, 
          synchronous: true
          archiveArtifacts allowEmptyArchive: true,
          artifacts: 'target/bom.xml', 
          fingerprint: true,
          onlyIfSuccessful: true
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
}