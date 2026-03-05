pipeline {
  agent {
    docker {
      // 1. Point this to the Java 21 image you just built and pushed
      image 'bharani301085/maven-jdk21-agent:v1' 
      args '--user root -v /var/run/docker.sock:/var/run/docker.sock'
    }
  }
  stages {
    stage('Checkout') {
      steps {
        sh 'echo "Checking out code..."'
        // Jenkins usually handles checkout automatically if this is a Pipeline project
      }
    }
    stage('Build and Test') {
      steps {
        sh 'ls -ltr'
        // This will now use JDK 21 from your custom image
        sh 'cd java-maven-sonar-argocd-helm-k8s/spring-boot-app && mvn clean package'
      }
    }
    stage('Static Code Analysis') {
      environment {
        SONAR_URL = "http://34.201.116.83:9000" // Verify your SonarQube IP is correct
      }
      steps {
        withCredentials([string(credentialsId: 'sonarqube', variable: 'SONAR_AUTH_TOKEN')]) {
          sh 'cd java-maven-sonar-argocd-helm-k8s/spring-boot-app && mvn sonar:sonar -Dsonar.login=$SONAR_AUTH_TOKEN -Dsonar.host.url=${SONAR_URL}'
        }
      }
    }
    stage('Build and Push Docker Image') {
      environment {
        // 2. Change this to YOUR Docker Hub username
        DOCKER_IMAGE = "bharani301085/spring-boot-app:${BUILD_NUMBER}"
      }
      steps {
        script {
            sh 'cd java-maven-sonar-argocd-helm-k8s/spring-boot-app && docker build -t ${DOCKER_IMAGE} .'
            // 3. Ensure "docker-cred" matches the ID you created in Jenkins Credentials
            docker.withRegistry('https://index.docker.io/v1/', "docker-cred") {
                docker.image("${DOCKER_IMAGE}").push()
            }
        }
      }
    }
    stage('Update Deployment File') {
        environment {
            // 4. Change these to YOUR GitHub details
            GIT_REPO_NAME = "YOUR_REPO_NAME_HERE"
            GIT_USER_NAME = "bharani301085" 
        }
        steps {
            withCredentials([string(credentialsId: 'github', variable: 'GITHUB_TOKEN')]) {
                sh '''
                    git config user.email "your-email@example.com"
                    git config user.name "bharani301085"
                    sed -i "s/replaceImageTag/${BUILD_NUMBER}/g" java-maven-sonar-argocd-helm-k8s/spring-boot-app-manifests/deployment.yml
                    git add java-maven-sonar-argocd-helm-k8s/spring-boot-app-manifests/deployment.yml
                    git commit -m "Update deployment image to version ${BUILD_NUMBER}"
                    git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
                '''
            }
        }
    }
  }
}