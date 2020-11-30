stage("Building Distributed Tasks") {
  jsTask("build-node", {
		checkout scm
    sh 'npm ci'

    sh "npx nx affected:test --all"
    sh "npx nx affected:lint --all"
    sh "npx nx affected:build --all"
  })
}

def jsTask(String label, Closure cl) {
	node(label) {
    withEnv(["PATH+NODEJS_HOME=${tool 'nodejs-10.23.0'}/bin"]) {
      cl()
    }
	}
}
