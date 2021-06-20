pipeline {
  agent none
  options {
     timeout(time: 1, unit: 'HOURS')
   }

  parameters {
    booleanParam(name: 'UNITTEST', defaultValue: true, description: 'Enable UnitTests ?')
	booleanParam(name: 'CODEANALYSIS', defaultValue: true, description: 'Enable CODE-ANALYSIS ?')
  }

  stages
 {
  stage('Checkout')
  {
       agent { label 'demo' }
       steps { 
          git branch: 'master', url: 'https://github.com/debasis4/cicd'
       }
  }

  stage('PreCheck')
  {
  agent { label 'demo' }
   steps {
       script {
          env.BUILDME = "yes" // Set env variable to enable further Build Stages
       }
   }
  }


  stage('Build Artifacts')
  {
   agent { label 'demo' }
   when {environment name: 'BUILDME', value: 'yes'}
   steps { 
     script {
	    if (params.UNITTEST) {
		  unitstr = ""
		} else {
		  unitstr = "-Dmaven.test.skip=true"
		}
	
		echo "Building Jar Component ..."
		dir ("./samplejar") {
		   sh "mvn clean package ${unitstr}"
		}

		echo "Building War Component ..."
		dir ("./samplewar") {
           sh "mvn clean package "
		}
	 }
   }
  }
  
  stage('Code Coverage')
  {
   agent { label 'demo' }
   when {
     allOf {
         expression { return params.CODEANALYSIS }
         environment name: 'BUILDME', value: 'yes'
     }
   }
   steps {
     echo "Running Code Coverage ..."  
	 
   }
  }
  
  stage('SonarQube Analysis') 
  {
    agent { label 'demo' }
    when {environment name: 'BUILDME', value: 'yes'}
    steps{
     echo "Running CODE ANALYSIS"
	  }
     }

  stage("Quality Gate"){ 
    when {environment name: 'BUILDME', value: 'yes'}
    steps{
	 echo  "Running Quality Gates"
    }
  }
}
}
