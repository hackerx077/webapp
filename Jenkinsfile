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
    
    stage ('Check-Git-Secrets') {
      steps {
        sh 'rm trufflehog || true'
        sh 'docker run gesellix/trufflehog --json https://github.com/hackerx077/webapp.git > trufflehog'
        sh 'cat trufflehog'
      }
    }
    
   stage ('Source-Composition-Analysis') {
		steps {
		     sh 'rm owasp-* || true'
		     sh 'wget https://raw.githubusercontent.com/hackerx077/webapp/master/owasp-dependency-check.sh'	
		     sh 'chmod +x owasp-dependency-check.sh'
		     sh 'bash owasp-dependency-check.sh'
		     sh 'cat /var/lib/jenkins/OWASP-Dependency-Check/reports/dependency-check-report.xml'
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
                sh 'scp -o StrictHostKeyChecking=no target/*.war kd@192.168.1.107:/prod/apache-tomcat-9.0.34/webapps/webapp.war'
              }      
           }       
    }
    
    stage ('Port Scan') {
	steps {
	sh 'rm nmap* || true'
	sh 'docker run --rm -v "$(pwd)":/data uzyexe/nmap -sS -sV -oX nmap 192.168.1.107'
	sh 'cat nmap'
 }
    }
	  
stage ('DAST') {
steps {
sshagent(['zap']) {
sh 'ssh -o StrictHostKeyChecking=no kd@192.168.1.103 "docker run -t owasp/zap2docker-stable zap-baseline.py -t http://192.168.1.107:8080/webapp/" || true'
	 }
}
	}
	  
stage ('Nikto Scan') {
steps {
sh 'rm nikto.xml || true'
sh 'docker pull secfigo/nikto:latest'
sh 'docker run -t secfigo/nikto secfigo/nikto.py -h http://192.168.1.107:8080'
sh 'cat target/nikto/nikto.xml'
	}
}
     stage ('SSL Checks') {
     steps {
     sh 'pip install sslyze==3.0.1'
     sh 'python -m sslyze --regular 8.8.8.8:8080 --json_out sslyze-output.json'
     sh 'cat sslyze-output.json'
		    }
	    }
  }
}
