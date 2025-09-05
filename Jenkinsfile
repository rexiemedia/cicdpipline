pipeline {
  agent any

  tools {
    // Correct the name to match what you have installed in Jenkins
    maven 'Maven 3.8.1' 
    jdk 'JDK 11'
  }

  // Define environment variables at the top level
  environment {
    SONAR_TOKEN_ID = 'sonar-token'
    NEXUS_CREDS_ID = 'nexus-creds'
    NEXUS_URL = 'http://192.168.1.100:8081/repository/maven-releases-test/'
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    // Wrap the subsequent stages in the 'dir' step
    // The 'dir' block is a step, not a stage.
    stage('Build and Analyze') {
      steps {
        dir('.') {
          sh 'mvn clean verify'
          withSonarQubeEnv('SonarQube') {
            sh "mvn sonar:sonar"
          }
        }
      }
    }

    stage('Quality Gate') {
      steps {
        timeout(time: 10, unit: 'MINUTES') {
          waitForQualityGate abortPipeline: true
        }
      }
    }

    stage('Deploy to Nexus') {
      steps {
        dir('.') {
          withCredentials([usernamePassword(credentialsId: "${env.NEXUS_CREDS_ID}", usernameVariable: 'NEXUS_USER', passwordVariable: 'NEXUS_PASS')]) {
            sh "mvn -B deploy -DaltDeploymentRepository=nexus::default::${env.NEXUS_URL}"
          }
        }
      }
    }
  }
}
