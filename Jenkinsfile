pipeline {
  agent any
  tools {
    // These names must exactly match the tool configurations in Jenkins Global Tool Configuration.
    maven 'Maven 3.8.6' 
    jdk 'JDK 11'
  }
  environment {
    // This variable holds the ID of the SonarQube token credential in Jenkins.
    SONAR_TOKEN_ID = 'sonar-token' 
    // This assumes your Nexus credentials ID is named 'nexus-creds'.
    NEXUS_CREDS_ID = 'nexus-creds'
    // This can be a variable or a hardcoded URL.
    NEXUS_URL = 'http://192.168.1.100:8081/repository/maven-releases-test/'
  }
  stages {
    stage('Checkout') {
      steps {
        // Use `checkout scm` to automatically check out the repository configured in the Jenkins job.
        checkout scm 
      }
    }
    stage('Build and Test') {
      steps {
        // `clean verify` compiles the code and runs all tests. It's a best practice to ensure the project is functional before proceeding.
        sh 'mvn clean verify' 
      }
    }
    stage('Code Analysis') {
      steps {
        // `withSonarQubeEnv` gets the SonarQube server details from Jenkins configuration.
        withSonarQubeEnv('SonarQube') {
          // `withCredentials` securely retrieves the SonarQube token and makes it available as an environment variable.
          withCredentials([string(credentialsId: "${env.SONAR_TOKEN_ID}", variable: 'SONAR_LOGIN')]) {
            sh "mvn sonar:sonar -Dsonar.login=${SONAR_LOGIN}"
          }
        }
      }
    }
    stage('Quality Gate') {
      steps {
        // The `waitForQualityGate` step pauses the pipeline until SonarQube's analysis is complete and its quality gate passes or fails. `abortPipeline: true` ensures the pipeline fails if the quality gate is not met.
        timeout(time: 5, unit: 'MINUTES') { 
          waitForQualityGate abortPipeline: true
        }
      }
    }
    stage('Deploy to Nexus') {
      steps {
        // `withCredentials` securely retrieves the Nexus username and password.
        withCredentials([usernamePassword(credentialsId: "${env.NEXUS_CREDS_ID}", usernameVariable: 'NEXUS_USER', passwordVariable: 'NEXUS_PASS')]) {
          // This command deploys the built artifact to Nexus. It uses environment variables to pass the credentials and the URL.
          sh "mvn -B deploy -DaltDeploymentRepository=nexus::default::${env.NEXUS_URL}"
        }
      }
    }
  }
}
