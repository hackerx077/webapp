pipeline {
  agent any 
  tools {
    maven 'Maven'
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
    
	  	  
      stage ('SAST') {
      steps {
        withSonarQubeEnv('sonar') {
          sh 'mvn sonar:sonar'
          sh 'cat target/sonar/report-task.txt'
        }
      }
    }
    
    stage ('Build') {
      steps {
      sh 'mvn clean package'
       }
    }
    
    stage ('Deploy-To-Tomcat') {
            steps {
           sshagent(['tomcat']) {
                sh 'scp -o StrictHostKeyChecking=no target/*.war kd@192.168.1.108:/prod/apache-tomcat-9.0.34/webapps/webapp.war'
              }      
           }       
    }
    
    stage ('Port Scan') {
	steps {
	sh 'rm nmap* || true'
	sh 'docker run --rm -v "$(pwd)":/data uzyexe/nmap -sS -sV -oX nmap 192.168.1.108'
	sh 'cat nmap'
 }
    }

stage ('Nikto Scan') {
steps {
sh 'docker pull secfigo/nikto:latest'
sh 'docker run -t secfigo/nikto secfigo/nikto.py -h http://192.168.1.108:8080'
	}
}

stage ('SSL Checks') {
steps {
sh 'docker run --rm zeitgeist/docker-sslscan www.google.com'
		    }
	    }
  
stage ('DAST') {	  
steps {
sshagent(['zap']) {
sh 'ssh -o StrictHostKeyChecking=no kd@192.168.1.105 "docker run -t owasp/zap2docker-stable zap-baseline.py -t http://192.168.1.108:8080/webapp/" || true'
 }
	}
}    	  
  
  }
}
