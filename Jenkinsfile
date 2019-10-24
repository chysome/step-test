pipeline {

    agent any 
    
    parameters {
      choice(name: 'applicationName', choices: ['Enterprise Census and Survey Enablement', 'LiMA/MCM', 'SQRA', 'CDL'], description: 'Name of application to build')
      
    }
   environment {
        DISABLE_AUTH = 'true'
        DB_ENGINE    = 'sqlite'
    }
   stages {
     stage('checkout scm') {
	 steps {		 
	        git branch: 'master',
		      credentialsId: 'svc-mjen-github',
			url: 'https://github.com/chysome/step-test.git'
	 	}
          }
	   
     stage("preserve build user") {
            wrap([$class: 'BuildUser']) {
                GET_BUILD_USER = sh ( script: 'echo "${BUILD_USER}"', returnStdout: true).trim()
            }
        }
     stage('Build') {
            steps {
                echo "Database engine is ${DB_ENGINE}"
                echo "DISABLE_AUTH is ${DISABLE_AUTH}"
		echo "The workspace is ${WORKSPACE}"
		echo " Approver is ${GET_BUILD_USER}"
                sh 'printenv'
            }
        }
       
     stage('log'){
	   steps {
		sh 'echo "foo" > a.log'
           }
       }
    
     stage('Example') {
         input {
                message "Press OK to continue"
                ok "Yes"
                submitter "Nkudu"
                parameters {
                    string(name: 'Approver', defaultValue: '', description: 'Deployment approver')
                }
         }
         when {
             beforeInput true
		 expression { params.applicationName == 'Enterprise Census and Survey Enablement'}
	     
             
         }
         options { 
             
             timeout(time: 3, unit: 'MINUTES') 
              
         }
         steps {
         echo "Good job Mr. ${Approver}, thanks for approving the deployment."
         echo "This is the real job id; ${BUILD_ID} and the job name is ${JOB_NAME}"
         echo "The build no is ${env.BUILD_NUMBER}"
	 archiveArtifacts artifacts: '*.log'
         }
       }
    stage('send email notification') { 
              steps { 
                  emailext ( 
                      subject: "Job '${env.JOB_NAME} ${env.BUILD_NUMBER}'", 
                      body: """CDL database and code deployment is in Progress, Check console output at "${env.BUILD_URL}" and monitor the console log. """, 
                      to: "eze@ezelxsvr.com", 
                     // from: "jenkinsezelxsvr.com" 
                      )
                    }
        }
	}
     
    post {
           always {
		    echo 'I will always run'
		    deleteDir()
                    //archiveArtifacts artifacts: '*.log'		        
           }
           aborted {
            echo 'I should be aborted if the pipeline was aborted'
           }
           failure {
               echo 'The pipeline had an epic fail'
               emailext (
                        subject: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
                        body: """<p>FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
                            <p>Check console output at &QUOT;<a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>&QUOT;</p>""",
                       // recipientProviders: [[$class: 'DevelopersRecipientProvider']]
                        to: 'ojukwu'
                        )
              
            }
           success {
            echo 'This would run only if successful'
           }
           fixed {
            echo 'The present execution passed, therefore I fixed the problem that failed the previous run'
           }
           regression {
            echo 'The pipeline failed, aborted or is unstable, therefore the previous run is better than me'
           }
           changed {
            //echo 'You will only see me because the present pipeline status changed from the previous one'
            echo '''
            This will run only if the state of the Pipeline has changed. 
	    For example, the Pipeline was previously failing but is now successful. 
            '''
           }
        
    }
}
