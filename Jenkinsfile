def distributedTasks = [: ]

pipeline {
  agent {
    label 'master-node'
  }
  options {
    skipDefaultCheckout true
  }
  stages {
    stage("Building Distributed Tasks") {
      steps {
        script {
          def checkoutCl = {
            stage('Checkout') {
              checkout scm
            }
          }
          def installCl = {
            stage('Install') {
              sh 'npm ci && zip --symlinks -r -q workspace.zip .'
              azureUpload storageCredentialId: 'microsoft-azure-storage',
              storageType: 'blobstorage',
              containerName: 'bc-quickstartjenkins',
              filesPath: 'workspace.zip',
              virtualPath: 'nx-jenkins-build'
            }
          }
          def runCl = {
            stage('Run') {
              distributedTasks << distributed('test', 3)
              distributedTasks << distributed('lint', 3)
              distributedTasks << distributed('build', 3)
            }
          }
          def cleanCl = {
            stage('Clean') {
              cleanWs()
            }
          }
          jsTask(checkoutCl, installCl, runCl, cleanCl)
        }
      }
    }
    stage("Run Distributed Tasks") {
      steps {
        script {
          parallel distributedTasks
        }
      }
    }
  }
}

def jsTask(Closure checkoutCl, Closure installCl, Closure runCl, Closure cleanCl) {
  node("build-node") {
    withEnv(["PATH+NODEJS_HOME=${tool 'nodejs-10.23.0'}/bin"]) {
      checkoutCl()
      installCl()
      runCl()
      cleanCl()
    }
  }
}

def distributed(String target, int bins) {
  def jobs = splitJobs(target, bins)
  def tasks = [: ]

  jobs.eachWithIndex {
    jobRun,
    i ->def list = jobRun.join(',')
    def title = "${target} - ${i}"

    def checkoutCl = {
      stage("${title} - Checkout") {
        azureDownload storageCredentialId: 'microsoft-azure-storage',
        downloadType: 'container',
        containerName: 'bc-quickstartjenkins',
        includeFilesPattern: 'nx-jenkins-build/workspace.zip',
        flattenDirectories: true
      }
    }
    def installCl = {
      stage("${title} - Install") {
        sh 'unzip -q workspace.zip'
      }
    }
    def runCl = {
      stage("${title} - Run") {
        sh "npx nx run-many --target=${target} --projects=${list}"
      }
    }
    def cleanCl = {
      stage("${title} - Clean") {
        cleanWs()
      }
    }

    tasks[title] = {
      jsTask(checkoutCl, installCl, runCl, cleanCl)
    }
  }

  return tasks
}

def splitJobs(String target, int bins) {
  def String baseSha = env.CHANGE_ID ? 'origin/master': 'origin/master~1'
  def String raw
  raw = sh(script: "npx nx print-affected --base=${baseSha} --target=${target}", returnStdout: true)
  def data = readJSON(text: raw)

  def tasks = data['tasks'].collect {
    it['target']['project']
  }

  if (tasks.size() == 0) {
    return tasks
  }

  // this has to happen because Math.ceil is not allowed by jenkins sandbox (╯°□°）╯︵ ┻━┻
  def c = sh(script: "echo \$(( ${tasks.size()} / ${bins} ))", returnStdout: true).toInteger()
  def split = tasks.collate(c)

  return split
}