pipeline{
    agent any
    environment {
            PROJECT_PATH = "/go/src/github.com/cjburchell/go-uatu"
    }

    stages {
        stage('Setup') {
            steps {
                script{
                    slackSend color: "good", message: "Job: ${env.JOB_NAME} with build number ${env.BUILD_NUMBER} started"
                }
             /* Let's make sure we have the repository cloned to our workspace */
             checkout scm
             }
         }

        stage('Lint') {
			agent {
				docker { 
					image 'cjburchell/goci:1.13'
					args '-v $WORKSPACE:$PROJECT_PATH'
				}
			}
            steps {
                script{
                        sh """go version"""
                        sh """printenv"""
                        sh """cd ${PROJECT_PATH} && ls"""
                        sh """stat /tmp || true"""
                        sh """cd /tmp && ls || true"""
                        sh """stat /tmp/.cache || true"""
                        sh """cd ${PROJECT_PATH} && go list ./..."""
                        sh """cd ${PROJECT_PATH} && go list ./... | grep -v /vendor/ > projectPaths"""
                        def paths = sh returnStdout: true, script:"""awk '{printf "/go/src/%s ",\$0} END {print ""}' projectPaths"""
                        sh """echo ${paths}"""

                        sh """go get github.com/nats-io/go-nats"""
                        sh """go get github.com/cjburchell/tools-go"""

                        sh """go vet ${paths}"""
                        sh """golint ${paths}"""

                        warnings canComputeNew: true, canResolveRelativePaths: true, categoriesPattern: '', consoleParsers: [[parserName: 'Go Vet'], [parserName: 'Go Lint']], defaultEncoding: '', excludePattern: '', healthy: '', includePattern: '', messagesPattern: '', unHealthy: ''
                }
            }
        }
    }

    post {
        always {
              script{
                  if ( currentBuild.currentResult == "SUCCESS" ) {
                    slackSend color: "good", message: "Job: ${env.JOB_NAME} with build number ${env.BUILD_NUMBER} was successful"
                  }
                  else if( currentBuild.currentResult == "FAILURE" ) {
                    slackSend color: "danger", message: "Job: ${env.JOB_NAME} with build number ${env.BUILD_NUMBER} was failed"
                  }
                  else if( currentBuild.currentResult == "UNSTABLE" ) {
                    slackSend color: "warning", message: "Job: ${env.JOB_NAME} with build number ${env.BUILD_NUMBER} was unstable"
                  }
                  else {
                    slackSend color: "danger", message: "Job: ${env.JOB_NAME} with build number ${env.BUILD_NUMBER} its result (${currentBuild.currentResult}) was unclear"
                  }
              }
        }
    }
}