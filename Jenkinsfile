pipeline {
  agent {
    label 'build-node'
  }
  options {
    skipDefaultCheckout true
  }
  tools {
    nodejs 'nodejs-10.23.0'
  }
  stages {
    stage("Checkout") {
      steps {
        checkout scm
      }
    }
    stage("Install") {
      steps {
        sh 'npm ci'
      }
    }
    stage("Test") {
      steps {
        sh "npx nx affected:test --all"
      }
    }
    stage("Lint") {
      steps {
        sh "npx nx affected:lint --all"
      }
    }
    stage("Build") {
      steps {
        sh "npx nx affected:build --all"
      }
    }
    stage("Clean") {
      steps {
        cleanWs()
      }
    }
  }
}