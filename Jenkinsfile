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
        sh 'docker run dxa4481/trufflehog --json https://github.com/redc0de-web/webapp.git > trufflehog'
        sh 'cat trufflehog'
      }
    }
    stage ('Source Composition Analysis') {
      steps {
         sh 'rm owasp* || true'
         sh 'wget "https://raw.githubusercontent.com/redc0de-web/webapp/master/owasp-dependency-check.sh" '
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
                sh 'scp -o StrictHostKeyChecking=no target/*.war ubuntu@3.12.34.209:/var/lib/tomcat8/webapps/webapp.war'
              }      
           }       
    }
    stage ('DAST') {
      steps {
        sshagent(['zap']) {
         sh 'ssh -o  StrictHostKeyChecking=no ubuntu@13.59.150.118 "docker run -t owasp/zap2docker-stable zap-baseline.py -t http://3.12.34.209:8080/webapp/" || true'
        }
      }
    }
    stage (''Notification mail'){
               mail bcc: '', body: '''Hi there ! this is your report 
               Thanks
               Naresh''', cc: '', from: '', replyTo: '', subject: 'jenkins Build Report ', to: 'dadp045@gmail.com'
             }
   } 
 }
