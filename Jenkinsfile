pipeline {
	agent {
    	node {
      		label 'master' 
   	 	}
  	}
  	tools { 
  		maven 'maven-3.5.4' 
        jdk 'jdk-8' 
    }
    stages {
        stage ('Initialize') {
            steps {
                sh '''
                    echo "PATH = ${PATH}"
                    echo "M2_HOME = ${M2_HOME}"
                '''
            }
        }
        stage('Checkout') {
            steps {
				checkout([$class: 'GitSCM', 
    		  		branches: [[name: '*/master']], 
    		  		doGenerateSubmoduleConfigurations: false, 
   			  		extensions: [[$class: 'CleanCheckout']], 
    		  		submoduleCfg: [], 
	    	  		userRemoteConfigs: [[url: 'https://github.com/ashwani-opstree/project-python-flask-webapp.git']]
           		])
            }
        }
        stage('SonarQube') {
            steps {
    			withSonarQubeEnv('sonarqube') {
					// some block
               	} 
           	}
      	}
		stage('BuildArtifact'){
			sh 'rm -rf /tmp/webapp.tar.gz; tar -czf /tmp/webapp.tar.gz .'
		}
        stage('Release') {
            steps {
				nexusPublisher nexusInstanceId: 'nexus3', nexusRepositoryId: 'py-release', packages: [[$class: 'MavenPackage', mavenAssetList: [[classifier: '', extension: '', filePath: '/tmp/webapp.tar.gz']], mavenCoordinate: [artifactId: 'fancyWidget', groupId: 'com.py', packaging: 'tar.gz', version: '1.2.5']]], tagName: 'build-125'				
            } 
        }
    }
}