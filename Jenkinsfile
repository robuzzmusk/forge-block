pipeline {
    agent {
        node {
            label 'maven'
        }
    }
environment {
    PATH = "/opt/apache-maven-3.9.9/bin:$PATH"
}    

    stages {
        stage('Clone-code') {
           steps {
               git branch: 'main', credentialsId: 'github-cred', url: 'https://github.com/robuzzmusk/forge-block.git'
           }
        }
        stage('build'){
            steps {
                echo "------------build started----------------"
                sh "mvn clean deploy -Dmaven.test.skip=true"
                echo "------------build completed------------"
            }
        }
        stage("test"){
            steps{
                echo "------------unit test started----------------"
                sh 'mvn surefire-report:report'
                echo "------------unit test completed------------"
            }
        }

        stage('SonarQube analysis') {
        environment {
          scannerHome = tool 'robuzz-sonar-scanner'
        }  
        steps { 
        withSonarQubeEnv('robuzz-sonarqube-server') { // If you have configured more than one global server connection, you can specify its name as configured in Jenkins
          sh "${scannerHome}/bin/sonar-scanner"
        }  
    }
  }

    stage("Quality Gate"){
            steps{
                script {
                timeout(time: 1, unit: 'HOURS') { // Just in case something goes wrong, pipeline will be killed after a timeout
           def qg = waitForQualityGate() // Reuse taskId previously collected by withSonarQubeEnv
           if (qg.status != 'OK') {
            error "Pipeline aborted due to quality gate failure: ${qg.status}"
    }
  }
}
}
}
}    
}