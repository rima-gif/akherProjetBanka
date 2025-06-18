pipeline {
  agent { label 'agent-node' }

tools {
  jdk 'Java 17'
  maven 'Maven'
}

  stages {

    stage("Cleanup Workspace") {
      steps {
        cleanWs()
      }
    }

    stage("Checkout Application Code") {
      steps {
        git(
          branch: 'main',
          credentialsId: 'github',
          url: 'https://github.com/rima-gif/akherProjetBanka.git'
        )
      }
    }

    stage("Build Application") {
      parallel {
        stage("SpringBoot") {
          steps {
            dir('ebanking-backend') {
              sh 'mvn clean install -DskipTests=true'
            }
          }
        }
        stage("Angular") {
          steps {
            dir('ebanking-frontend') {
              sh 'npm install'
              sh './node_modules/.bin/ng build --configuration production'
            }
          }
        }
      }
    }
    stage("SonarQube Analysis") {
      steps {
        dir('ebanking-backend') {
          withSonarQubeEnv(installationName: 'sonarqube-server',credentialsId:'jenkins-sonarqube-token') {
            sh "mvn sonar:sonar"
          }
        }
      }
    }
    stage("SonarQube Analysis - Angular") {
  steps {
    dir('ebanking-frontend') {
      withSonarQubeEnv(installationName: 'sonarqube-server', credentialsId: 'jenkins-sonarqube-token') {
        sh 'sonar-scanner'
      }
    }
  }
}

  }
}
