def registry = 'https://robuzz.jfrog.io'
def imageName = 'robuzz.jfrog.io/galaxy-docker-local/ttrend'
def version   = '2.1.2'
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
        // stage("test"){
        //     steps{
        //         echo "------------unit test started----------------"
        //         sh 'mvn surefire-report:report'
        //         echo "------------unit test completed------------"
        //     }
        // }

//         stage('SonarQube analysis') {
//         environment {
//           scannerHome = tool 'sonar-scanner'
//         }  
//         steps { 
//         withSonarQubeEnv('sonarqube-server') { // If you have configured more than one global server connection, you can specify its name as configured in Jenkins
//           sh "${scannerHome}/bin/sonar-scanner"
// sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=robunk-key_twittertrend -Dsonar.sources=. -Dsonar.host.url=http://your_sonarqube_server -Dsonar.login=e270445dd707622d969d4f8685396a55aae6103b"
//         }  
//     }
//   }

//     stage("Quality Gate"){
//             steps{
//                 script {
//                 timeout(time: 1, unit: 'HOURS') { // Just in case something goes wrong, pipeline will be killed after a timeout
//            def qg = waitForQualityGate() // Reuse taskId previously collected by withSonarQubeEnv
//            if (qg.status != 'OK') {
//             error "Pipeline aborted due to quality gate failure: ${qg.status}"
//     }
//   }
// }
// }
// }
    stage("Jar Publish") {
        steps {
            script {
                    echo '<--------------- Jar Publish Started --------------->'
                     def server = Artifactory.newServer url:registry+"/artifactory" ,  credentialsId:"artifact-cred"
                     def properties = "buildid=${env.BUILD_ID},commitid=${GIT_COMMIT}";
                     def uploadSpec = """{
                          "files": [
                            {
                              "pattern": "jarstaging/(*)",
                              "target": "libs-libs-release-local/{1}",
                              "flat": "false",
                              "props" : "${properties}",
                              "exclusions": [ "*.sha1", "*.md5"]
                            }
                         ]
                     }"""
                     def buildInfo = server.upload(uploadSpec)
                     buildInfo.env.collect()
                     server.publishBuildInfo(buildInfo)
                     echo '<--------------- Jar Publish Ended --------------->'  
            
            }
        }   
    }

       
    stage(" Docker Build ") {
      steps {
        script {
           echo '<--------------- Docker Build Started --------------->'
           app = docker.build(imageName+":"+version)
           echo '<--------------- Docker Build Ends --------------->'
        }
      }
    }

    stage (" Docker Publish "){
        steps {
            script {
               echo '<--------------- Docker Publish Started --------------->'  
                docker.withRegistry(registry, 'artifact-cred'){
                    app.push()
                }    
               echo '<--------------- Docker Publish Ended --------------->'  
            }
        }
    }   
}    
}