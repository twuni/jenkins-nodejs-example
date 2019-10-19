pipeline {
  agent none

  stages {
    stage('initialize') {
      agent any

      steps {
        checkout scm
      }
    }

    stage('install') {
      agent any

      steps {
        sh 'yarn install --frozen-lockfile'
        stash includes: 'node_modules', name: 'dependencies'
      }
    }

    parallel {
      stage('build') {
        agent any

        environment {
          NODE_ENV = 'production'
        }

        steps {
          unstash 'dependencies'
          sh 'yarn build'
        }
      }

      stage('lint') {
        agent any

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
        agent any

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
        agent any

        steps {
          unstash 'dependencies'
          sh 'yarn --silent documentation'
        }

        post {
          always {
            archiveArtifacts artifacts: 'docs', fingerprint: true
          }
        }
      }
    }

    stage('publish') {
      agent any

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
