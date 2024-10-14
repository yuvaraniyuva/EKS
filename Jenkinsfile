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
      
      stage('Build & Push Docker Image') {
            steps {
                script {
                  dockerImage = docker.build("devopsyuvi/tomcat:latest")
                  //docker.withRegistry( '', DOCKER_LOGIN ){
                  //dockerImage.push("$BUILD_NUMBER")
                  //dockerImage.push()
                  withCredentials([string(credentialsId: DOCKER_LOGIN, variable: 'DOCKER_TOKEN')]) {
                        sh 'echo $DOCKER_TOKEN | docker login -u your-docker-username --password-stdin'
                        docker.withRegistry('', "${DOCKER_TOKEN}") {
                            docker.image("${DOCKER_IMAGE}").push()
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
