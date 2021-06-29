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
    git branch: 'master', url: 'https://gitlab.com/debasis4/cicd.git'
   }
  }
  
  // Sets a env variable based on commits made on samplejar/war dir //
  stage('PreCheck')
  {
  agent { label 'demo' }
   when { 
     anyOf {
           changeset "samplejar/**"
           changeset "samplewar/**"
     }
   }
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
	 dir ("./samplejar") {
	   sh "mvn  org.jacoco:jacoco-maven-plugin:0.5.5.201112152213:prepare-agent"
	 }
   }
  }
  
  stage('Stage Artifacts') 
  {          
   agent { label 'demo' }
   when {environment name: 'BUILDME', value: 'yes'}
   steps {          
    script { 
	    /* Define the Artifactory Server details */
        def server = Artifactory.server 'myartifactory'
        def uploadSpec = """{
            "files": [{
            "pattern": "samplewar/target/samplewar.war", 
            "target": "test"                   
            }]
        }"""
        
        /* Upload the war to  Artifactory repo */
        server.upload(uploadSpec)
    }
   }
  }
  
  stage('SonarQube Analysis') 
  {
    agent { label 'demo' }
    when {environment name: 'BUILDME', value: 'yes'}
    steps{
     withSonarQubeEnv('mysonarqube') {
	  dir ("./samplejar") {
         sh 'mvn sonar:sonar'
	  }
     } 
    }
  }
  
  stage("Quality Gate"){ 
    when {environment name: 'BUILDME', value: 'yes'}
    steps{
	 script {
	  timeout(time: 10, unit: 'MINUTES') { // Just in case something goes wrong, pipeline will be killed after a timeout
        def qg = waitForQualityGate() // Reuse taskId previously collected by withSonarQubeEnv
        if (qg.status != 'OK') {
           error "Pipeline aborted due to quality gate failure: ${qg.status}"
        }
      }
	 }
    }
  }
  
   stage('Build Image') 
  {
    agent { label 'demo' }
    when {environment name: 'BUILDME', value: 'yes'}
    steps{
      script {
          docker.withRegistry( 'https://registry.hub.docker.com', 'dockerhub' ) {
             /* Build Docker Image locally */
             myImage = docker.build("debasis4/democicd:latest")

             /* Push the container to the Registry */
             myImage.push()
          }
      }
    }
  }


   stage('Deploy')
   {
       agent { label 'deploy' }
       when {environment name: 'BUILDME', value: 'yes'}
	   steps {
	    echo "Checkout Deployment configuration files .."
	    git branch: 'master', url: 'https://gitlab.com/debasis4/cicd.git'
	    step([$class: 'DockerComposeBuilder', dockerComopseFile: 'docker-compose.yml' , option: [$class:]) 
		}
	   }
	}
	
  stage('Smoke Test')
  {
    agent { label 'demo' }
    when {environment name: 'BUILDME', value: 'yes'}
    steps {
      sh "chmod +x runsanity.sh; ./runsanity.sh debasis4/democicd:latest"
    }
  }
  
 } //End of Stages

} //End of Pipeline

