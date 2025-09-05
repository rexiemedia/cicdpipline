pipeline {
  agent any
  tools {
    maven 'Maven 3.8.1'
  }
  environment {
    SONAR_TOKEN = credentials('sonar-token-id') // Add via Jenkins Credentials
  }
  stages {
    stage('Checkout') {
      steps {
        git 'https://github.com/yourusername/yourrepo.git'
      }
    }
    stage('Build') {
      steps {
        sh 'mvn clean install'
      }
    }
    stage('Code Analysis') {
      steps {
        withSonarQubeEnv('SonarQube') {
          sh 'mvn sonar:sonar -Dsonar.login=$SONAR_TOKEN'
        }
      }
    }
    stage('Deploy to Nexus') {
      steps {
        sh 'mvn deploy'
      }
    }
  }
}
