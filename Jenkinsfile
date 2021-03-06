pipeline {
    agent any
	
	tools{
		
	    maven 'maven'
	    jdk  'jdk'
	}
    options {
    // Only keep the 10 most recent builds
    buildDiscarder(logRotator(numToKeepStr:'10'))
    retry(3) 
  }
   

    parameters {
      choice(name: 'applicationName', choices: ['LiMA/MCM', 'Enterprise Census and Survey Enablement', 'SQRA', 'CDL'], description: 'Name of application to build')
    }
    environment {
        DISABLE_AUTH = 'true'
        DB_ENGINE    = 'sqlite'
	//SSH_CREDS = credentials('svc-mjen-github-ssh')
    }
    stages {    
	    
	stage ('Initialize') {
            steps {
                sh '''
                    echo "PATH = ${PATH}"
                    echo "MAVEN_HOME = ${MAVEN_HOME}"
                ''' 
            }
        }
	
	    
        stage('Build') {
		
            steps {
                echo "Database engine is ${DB_ENGINE}"
                echo "DISABLE_AUTH is ${DISABLE_AUTH}"
                echo "The workspace is ${WORKSPACE}"
                sh 'printenv'
            }
        }        
        stage('log') {
            steps {
            sh 'echo "foo" > a.log'
            }
        }        
        stage('Example') {
            input {
                message "Press OK to continue"
                ok "Yes"
                submitter "David"
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
                echo "This is the real build number; ${env.BUILD_ID} and the job name is ${env.JOB_NAME} and ${env.DB_ENGINE}"
                echo "The build no is ${env.BUILD_NUMBER}"
                
            }
        }
        stage('send email notification') { 
            steps { 
                emailext ( 
                    subject: "Job Name '${env.JOB_NAME} and Build number ${env.BUILD_NUMBER}'", 
                    body: """EcaSE FOCS deployment is in Progress, Check console output at "${env.BUILD_URL}" and monitor the console log""", 
                    to:   "eze@ezelxsvr.com" 
                    
                )
            }
        }
    }
     
    post {
        always {
		    echo 'I will always run'
		    //deleteDir()
            archiveArtifacts artifacts: '*.log'		        
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
                to:     "eze@ezelxsvr.com"
            )
              
        }
        success {
            echo 'This would run only if successful'
            emailext (
                subject: "For your information",
                body: "${currentBuild.fullDisplayName} has succeeded",
                to: "eze@ezelxsvr.com"               
            )
        }
        fixed {
            echo 'The present execution passed, therefore I fixed the problem that failed the previous run'
        }
        regression {
            echo 'The pipeline failed, aborted or is unstable, therefore the previous run is better than me'
        }
        changed {
            echo 'You will only see me because the present pipeline status changed from the previous one'
        }
        
    }
}
