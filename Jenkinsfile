pipeline{
    agent { label 'tomcat' } 
    tools {
        maven "Maven3"
        jdk "java17"
    }

    environment{
      NEXUS_USER = 'admin'
      NEXUS_PASS = 'password'
      RELEASE_REPO = 'artifact-upload'
      CENTRAL_REPO = 'maven-dep'
      NEXUSIP = '172.31.21.44'
      NEXUSPORT = '8081'
      NEXUS_GRP_REPO = 'group'
      NEXUS_LOGIN = 'nexuslogin'
      DOCKER_LOGIN = 'dockerlogin'
      DOCKER_TOKEN = 'dockertoken'
      SONARSERVER = 'sonarserver'
      SONARSCANNER = 'sonarscanner'
    }
    
    stages {
      stage('Build'){
        steps{
          sh 'mvn -s settings.xml -DskipTests install'
        }
        post{
          success{
            echo "Now Archiving"
            archiveArtifacts artifacts: '**/*.war'
          }
        }
      }

      stage('Test'){
        steps{
          sh 'mvn -s settings.xml test'
        }
      }  

      stage('Checkstyle Analysis'){
        steps{
          sh 'mvn -s settings.xml checkstyle:checkstyle'
        }
      }

      stage('Sonar Analysis') {
        environment {
          scannerHome = tool "${SONARSCANNER}"
        }
        steps {
          withSonarQubeEnv("${SONARSERVER}") {
                   sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                   -Dsonar.projectName=vprofile \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
          }
        }
      }

      stage("Quality Gate") {
          steps {
            timeout(time: 5, unit: 'MINUTES') {
                    // Parameter indicates whether to set pipeline to UNSTABLE if Quality Gate fails
                    // true = set pipeline to UNSTABLE, false = don't
              waitForQualityGate abortPipeline: true
                }
            }
      }

      stage("UploadArtifact"){
        steps{
          nexusArtifactUploader(
            nexusVersion: 'nexus3',
            protocol: 'http',
            nexusUrl: "${NEXUSIP}:${NEXUSPORT}",
            groupId: 'Build_Artifact',
            version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}",
            repository: "${RELEASE_REPO}",
            credentialsId: "${NEXUS_LOGIN}",
            artifacts: [
              [artifactId: 'CICD',
              classifier: '',
              file: 'target/vprofile-v2.war',
              type: 'war']
            ]
          )
        }
      } 
      
      //stage('Build & Push Docker Image') {
      //      steps {
    //          script {
       //           dockerImage = docker.build("devopsyuvi/tomcat:latest")
        //          docker.withRegistry( '', DOCKER_LOGIN ){
        //          //dockerImage.push("$BUILD_NUMBER")
        //          dockerImage.push()
         //         }

                  
  //              }
   //         }
   //   }


      stage('Build & Push Docker Image') {
            steps {
                script {
                  dockerImage = docker.build( "devopsyuvi/tomcat:$BUILD_NUMBER")
                  withCredentials([string(credentialsId: DOCKER_TOKEN, variable: 'DOCKER_TOKEN')]) {
                        sh 'echo $DOCKER_TOKEN | docker login -u devopsyuvi --password-stdin'
                        docker.withRegistry('', DOCKER_LOGIN) {
                            dockerImage.push("$BUILD_NUMBER")
                            dockerImage.push('latest')
                        }
                  }
                }

                  
            }
      }
      stage('Deploy to EKS') {
            steps {
                script {
                    sh """
                        kubectl apply -f deployment.yaml
                        kubectl apply -f service.yaml
                    """
                }
            }
      }
    
    }
}
