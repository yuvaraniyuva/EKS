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
      SONARSERVER = 'sonarserver'
      SONARSCANNER = 'sonarscanner'
    }
    
    stages {
      
      stage('Build & Push Docker Image') {
            steps {
                script {
                  dockerImage = docker.build("tomcat:latest")
                  docker.withRegistry( '', DOCKER_LOGIN ){
                  //dockerImage.push("$BUILD_NUMBER")
                  dockerImage.push()
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
