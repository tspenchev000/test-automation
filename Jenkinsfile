def git_url = 'https://github.com/tspenchev000/test-automation.git'
def Branch = "unknown"

pipeline {
    agent any;
    options {
        // values: NANOSECONDS, MICROSECONDS, MILLISECONDS, SECONDS, MINUTES, HOURS, DAYS
        timeout(time: 5, unit: 'MINUTES') 
        buildDiscarder(logRotator(numToKeepStr: '20', artifactNumToKeepStr: '20'))
        disableConcurrentBuilds()
    }	
    stages {
        stage('Nothing interesting') {
            steps {
                script {
                    sh """
                      ls -al
                      whoami
                      java -version
                    """
                }
            }
        }        
		stage('Action') {
			steps {
			    script {
                    Branch = "uat"
                }
            }
        }
        stage('Checkout') {
            steps {
                git(
                    branch: "${Branch}",
                    url: git_url
                )
            }
        }

		stage('End') {
			steps {
			    script {
                    sh 'pwd'
                    sh 'echo hello'
			    }
			}
   }
  } // stages
} // pipeline
