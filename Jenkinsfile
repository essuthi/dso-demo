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
            container('maven') {
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
            container('maven') {
              sh 'mvn test'
            }
          }
        }

        stage('SCA') {
          steps{
            container('maven'){
              catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                sh 'mvn org.owasp:dependency-check-maven:check'
              }
            }
          }
          post {
            always {
              archiveArtifacts allowEmptyArchive: true, artifacts: 'target/dependency-check-report.html', fingerprint:true, onlyIfSuccessful: true
              //dependencyCheckPublisher pattern: 'report.xml'
            }
          }
        }
        stage('OSS License Checker') {
          steps{
            container('licensefinder') {
              sh 'ls -al'
              sh '''
                #!/bin/bash--login
                /bin/bash--login
                rvm use default
                gem install license_finder
                license_finder
              '''
            }
          }
        }
        stage('SAST') {
          steps{
            container('slscan') {
              sh 'scan --type java,depscan --buid'
            }
          }
          post {
            success {
              archiveArtifacts allowEmptyArchive: true, artifacts: 'reports/*', fingerprint: true, onlyIfSuccessful: true
            }
          }
        }
      }
    }
    stage('Package') {
      parallel {
        stage('Create Jarfile') {
          steps {
            container('maven') {
              sh 'mvn package -DskipTests'
            }
          }
        }

        stage('OCI Image BnP') {
          steps{
            container('kaniko') {
              sh '''
                /kaniko/executor -f `pwd`/Dockerfile -c `pwd` --insecure --skip-tls-verify --cache=true --destination=docker.io/essuthiessoh/dso-demo
              '''
            }
          }
        }
      }
    }

    stage('Deploy to Dev') {
      steps {
        // TODO
        sh "echo done"
      }
    }
  }
}
