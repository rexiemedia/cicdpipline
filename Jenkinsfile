pipeline {
  agent any

  tools {
    maven 'Maven 3.8.6'       // Matches your configured Maven version
    jdk 'JDK 11'              // Java version for compilation
    sonarQubeScanner 'SonarScanner' // Required for SonarQube analysis
  }

  environment {
    SONAR_TOKEN_ID = 'sonar-token'         // Jenkins credential ID for SonarQube token
    NEXUS_CREDS_ID = 'nexus-creds'         // Jenkins credential ID for Nexus
    NEXUS_URL = 'http://192.168.1.100:8081/repository/maven-releases-test/' // Nexus repo URL
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build and Test') {
      steps {
        sh 'mvn clean verify'
      }
    }

    stage('SonarQube Analysis') {
      steps {
        withSonarQubeEnv('SonarQube') {
          withCredentials([string(credentialsId: "${env.SONAR_TOKEN_ID}", variable: 'SONAR_LOGIN')]) {
            sh "mvn sonar:sonar -Dsonar.login=${SONAR_LOGIN}"
          }
        }
      }
    }

    stage('Quality Gate') {
      steps {
        timeout(time: 5, unit: 'MINUTES') {
          waitForQualityGate abortPipeline: true
        }
      }
    }

    stage('Deploy to Nexus') {
      steps {
        withCredentials([usernamePassword(credentialsId: "${env.NEXUS_CREDS_ID}", usernameVariable: 'NEXUS_USER', passwordVariable: 'NEXUS_PASS')]) {
          sh "mvn -B deploy -DaltDeploymentRepository=nexus::default::${env.NEXUS_URL}"
        }
      }
    }
  }
}
