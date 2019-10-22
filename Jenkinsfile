pipeline {
  agent {
    docker {
      image 'twuni/nodejs:12.12.0'
    }
  }

  stages {
    stage('initialize') {
      steps {
        checkout scm
      }
    }

    stage('install') {
      steps {
        sh 'yarn install --frozen-lockfile'
        stash includes: 'node_modules', name: 'dependencies'
      }
    }

    stage('parallel') {
      parallel {
        stage('build') {
          environment {
            NODE_ENV = 'production'
          }

          steps {
            unstash 'dependencies'
            sh 'yarn build'
          }
        }

        stage('lint') {
          steps {
            unstash 'dependencies'
            sh 'yarn --silent lint --format junit --output-file eslint.xml'
          }

          post {
            always {
              junit 'eslint.xml'
            }
          }
        }

        stage('test') {
          steps {
            unstash 'dependencies'
            sh 'yarn --silent test --reporter xunit > junit.xml'
          }

          post {
            always {
              junit 'junit.xml'
            }
          }
        }

        stage('documentation') {
          steps {
            unstash 'dependencies'
            sh 'yarn --silent documentation'
          }

          post {
            success {
              archiveArtifacts artifacts: 'docs', fingerprint: true
            }
          }
        }
      }
    }

    stage('publish') {
      environment {
        AWS_ACCESS_KEY_ID = credentials('AWS_ACCESS_KEY_ID')
        AWS_REGION = credentials('AWS_REGION')
        AWS_SECRET_ACCESS_KEY = credentials('AWS_SECRET_ACCESS_KEY')
      }

      steps {
        echo 'Publishing...'
      }
    }
  }
}
