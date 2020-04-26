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
	  
stage ('Nikto Scan') {
steps {
sh 'docker pull secfigo/nikto:latest'
sh 'docker run -t secfigo/nikto secfigo/nikto.py -h http://192.168.1.107:8080'
	}
}
     stage ('SSL Checks') {
     steps {
     sh 'pip install sslyze'
     sh 'python -m sslyze --regular 8.8.8.8:8080 --json_out sslyze-output.json'
     sh	'python -m sslyze --regular www.google.com --json_out=results.json'
     sh 'cat sslyze-output.json'
		    }
	    }
  }
}
