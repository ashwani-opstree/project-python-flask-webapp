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
        stage('SonarQube analysis') {
			steps {
                script {
             		scannerHome = tool 'sonarscanner';
        		}
    			withSonarQubeEnv('sonarqube') {
					sh "${scannerHome}/bin/sonar-scanner"
               	} 
           	}
      	}
		stage('BuildArtifact'){
			steps {
				sh 'rm -rf /tmp/webapp.tar.gz; tar -czf /tmp/webapp.tar.gz .'
			}
		}
        stage('Release') {
            steps {
				nexusPublisher nexusInstanceId: 'nexus3', 
				nexusRepositoryId: 'py-release', 
				packages: [[
					$class: 'MavenPackage', 
					mavenAssetList: [[
						classifier: '', 
						extension: '', 
						filePath: '/tmp/webapp.tar.gz'
					]], 
					mavenCoordinate: [
						artifactId: 'webapp', 
						groupId: 'com.py', 
						packaging: 'tar.gz', 
						version: '2.9.0'
					]
				]]				
            } 
        }
		stage('Deploy App'){
			steps {
				sh '''
				    kill -9 `cat /tmp/pidfile` || true
					rm -rf /tmp/app
				    mkdir -p /tmp/app
					curl -u admin:admin123 -X GET 'http://18.210.172.8:8081/repository/py-release/com/py/webapp/2.4.0/webapp-2.4.0.tar.zip' -o /tmp/app/webapp.tar.gz
					cd /tmp/app; tar -xvf webapp.tar.gz
					cd /tmp/app; BUILD_ID=dontKillMe python runserver.py > /tmp/app.log 2>&1 & echo $! > /tmp/pidfile
				'''
			}
		}
    }
}