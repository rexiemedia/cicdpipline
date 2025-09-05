pipeline {
  agent any

  tools {
    // These names must match the names in Global Tool Configuration.
    maven 'Maven 3.8.6' 
    jdk 'JDK 11'
  }

  environment {
    // The credential IDs configured in Jenkins.
    SONAR_TOKEN_ID = 'sonar-token' 
    NEXUS_CREDS_ID = 'nexus-creds'
    // This is the URL of your Nexus repository.
    NEXUS_URL = 'http://192.168.1.100:8081/repository/maven-releases-test/'
  }

  stages {
    stage('Checkout') {
      steps {
        // This step checks out the code into the build agent's workspace.
        checkout scm
      }
    }

    // All subsequent Maven commands should be run from the project directory.
    // The `dir` step changes the working directory.
    dir('.') {
      stage('Build and Test') {
        steps {
          sh 'mvn clean verify'
        }
      }

      stage('SonarQube Analysis') {
        steps {
          // This step automatically injects the server URL and authentication token configured in Jenkins.
          withSonarQubeEnv('SonarQube') {
            sh "mvn sonar:sonar"
          }
        }
      }
    } // End of `dir('.')` block

    stage('Quality Gate') {
      steps {
        timeout(time: 5, unit: 'MINUTES') {
          waitForQualityGate abortPipeline: true
        }
      }
    }

    stage('Deploy to Nexus') {
      steps {
        // The `dir` step here is also crucial to ensure `mvn deploy` runs from the correct location.
        dir('.') {
          withCredentials([usernamePassword(credentialsId: "${env.NEXUS_CREDS_ID}", usernameVariable: 'NEXUS_USER', passwordVariable: 'NEXUS_PASS')]) {
            sh "mvn -B deploy -DaltDeploymentRepository=nexus::default::${env.NEXUS_URL}"
          }
        }
      }
    }
  }
}
