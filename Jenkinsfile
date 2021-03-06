pipeline {
	agent none


	/*options {
		checkoutToSubdirectory("/var/www/")
	}*/

	stages {

		stage('Pre-Build') {
			agent any
			steps {
				sh '''
					printenv
					ls -la /var/
					echo $(date)

				'''
			}
		}

		stage("Build") {
			agent any
			steps {
				echo "Building...${env.BUILD_ID}" 
			}
		}

		stage("Build on docker image") {
			agent {
				docker {
					image 'node:10-alpine'
					args '-u root -p 3000:3000'
				}
			}
			environment {
				CI = 'true'
			}
			steps {
				sh '''
					npm install
					./jenkins/scripts/test.sh
				'''
			}
		}

		stage("Test") {
			agent any
			steps {
				echo "Testing...${env.BUILD_ID}"
			}
		}

		stage("Deploy") {
			agent any
			steps {
				sh './jenkins/scripts/deliver.sh' 
			}
		}

	}

	post {
        failure {
            echo 'Sending email...'
            
            emailext body: "${currentBuild.currentResult}: Job ${env.JOB_NAME} build ${env.BUILD_NUMBER}\n More info at: ${env.BUILD_URL}",
		                recipientProviders: [
		                	[$class: 'DevelopersRecipientProvider'], 
		                	[$class: 'RequesterRecipientProvider']
		                ],
		                subject: "Jenkins Build ${currentBuild.currentResult}: Job ${env.JOB_NAME}"
            
        }
    }
}