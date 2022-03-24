pipeline {
	    agent any
	

	        // Environment Variables
	        environment {
	        MAJOR = '1'
	        MINOR = '1'
	        //Orchestrator Services
	        UIPATH_ORCH_URL = "https://cloud.uipath.com/"
	        UIPATH_ORCH_LOGICAL_NAME = "psrmbeiurq"
	        UIPATH_ORCH_TENANT_NAME = "Default"
	        UIPATH_ORCH_FOLDER_NAME = "Default"
	    }
	

	    stages {
	

	        // Printing Basic Information
	        stage('Preparing'){
	            steps {
	                echo "Jenkins Home ${env.JENKINS_HOME}"
	                echo "Jenkins URL ${env.JENKINS_URL}"
	                echo "Jenkins JOB Number ${env.BUILD_NUMBER}"
	                echo "Jenkins JOB Name ${env.JOB_NAME}"
	                echo "GitHub BranchName ${env.BRANCH_NAME}"
	                checkout scm
	

	            }
	        }
	

	         // Building Tests
	        stage('Build Tests') {
	            steps {
	                echo "Building package with ${WORKSPACE}"
	                UiPathPack (
	                      outputPath: "Output\\Tests\${env.BUILD_NUMBER}",
						  outputType: 'Tests',
	                      projectJsonPath: "project.json",
	                      version: [$class: 'ManualVersionEntry', version: "${MAJOR}.${MINOR}.${env.BUILD_NUMBER}"],
	                      useOrchestrator: false,
						  traceLevel: 'None'
						)
	            }
	        }
			
	         // Deploy Stages
	        stage('Deploy Tests') {
	            steps {
	                echo "Deploying ${BRANCH_NAME} to orchestrator"
	                UiPathDeploy (
	                packagePath: "Output\\Tests\${env.BUILD_NUMBER}",
	                orchestratorAddress: "${UIPATH_ORCH_URL}",
	                orchestratorTenant: "${UIPATH_ORCH_TENANT_NAME}",
	                folderName: "${UIPATH_ORCH_FOLDER_NAME}",
	                environments: 'INT',
	                //credentials: [$class: 'UserPassAuthenticationEntry', credentialsId: 'APIUserKey']
	                credentials: Token(accountName: "${UIPATH_ORCH_LOGICAL_NAME}", credentialsId: 'APIUserKey'),
					traceLevel: 'None',
					entryPointPaths: 'Main.xaml'
	

					)
	            }
			
			}
			
			
	         // Email Notification before PROD Deployment
 	        stage('Email Notification') {
 	            steps {
 	                echo 'Testing the workflow...'
			mail bcc: '', body: '''Hi,

			Process Test case deployed and Production Environment deployment started.

			$PROJECT_NAME - Build # $BUILD_NUMBER - $BUILD_STATUS:
			Check console output at $BUILD_URL to view the results.

			Thanks and Regards,
			S.Purushothaman
			RPA Developer''', cc: '', from: '', replyTo: '', subject: 'Jenkins Process Deployment Notification', to: 'srmp24@gmail.com'
					
 	            }
 			}
			
		stage('Approval') {
            	    // no agent, so executors are not used up when waiting for approvals
                    steps {
                        script {
                            def deploymentDelay = input id: 'Deploy', message: 'Deploy to production?', submitter: 'rkivisto,admin', parameters: [choice(choices: ['0', '1', '2', '3', '4', '5', '6', '7', '8', '9', '10', '11', '12', '13', '14', '15', '16', '17', '18', '19', '20', '21', '22', '23', '24'], description: 'Hours to delay deployment?', name: 'deploymentDelay')]
                            sleep time: deploymentDelay.toInteger(), unit: 'HOURS'
                }
            }
        }
	         // Building Package
	        stage('Build Process') {
				when {
					expression {
						currentBuild.result == null || currentBuild.result == 'SUCCESS'
						}
				}
				steps {
					echo "Building package with ${WORKSPACE}"
					UiPathPack (
						  outputPath: "Output\\${env.BUILD_NUMBER}",
						  projectJsonPath: "project.json",
						  version: [$class: 'ManualVersionEntry', version: "${MAJOR}.${MINOR}.${env.BUILD_NUMBER}"],
						  useOrchestrator: false,
						  traceLevel: 'None'
						)
					}
	        }			
			
	         // Deploy to Production Step
	        stage('Deploy Process') {
				when {
					expression {
						currentBuild.result == null || currentBuild.result == 'SUCCESS' 
						}
				}
				steps {
	                echo 'Deploying process to orchestrator...'
	                UiPathDeploy (
	                packagePath: "Output\\${env.BUILD_NUMBER}",
	                orchestratorAddress: "${UIPATH_ORCH_URL}",
	                orchestratorTenant: "${UIPATH_ORCH_TENANT_NAME}",
	                folderName: "${UIPATH_ORCH_FOLDER_NAME}",
	                environments: 'INT',
	                //credentials: [$class: 'UserPassAuthenticationEntry', credentialsId: 'APIUserKey']
	                credentials: Token(accountName: "${UIPATH_ORCH_LOGICAL_NAME}", credentialsId: 'APIUserKey'),
					traceLevel: 'None',
					entryPointPaths: 'Main.xaml'
					)
				}   
			}	
		
	    }
	

	    // Options
	    options {
	        // Timeout for pipeline
	        timeout(time:80, unit:'MINUTES')
	        skipDefaultCheckout()
	    }
	

	

	    // 
	    post {
	        success {
	            echo 'Deployment has been completed!'
	        }
	        failure {
	          echo "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.JOB_DISPLAY_URL})"
	        }
	        always {
	            /* Clean workspace if success */
	            cleanWs()
	        }
	    }
	
	}
		
	


