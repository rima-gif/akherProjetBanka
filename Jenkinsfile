pipeline {
  agent { label 'agent-node' }

  tools {
    jdk 'Java 17'
    maven 'Maven'
    // sonarqube-scanner doit être configuré dans Jenkins Tools (Global Tool Configuration)
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
          withSonarQubeEnv(installationName: 'sonarqube-server', credentialsId: 'jenkins-sonarqube-token') {
            sh "mvn sonar:sonar"
          }
        }
      }
    }

    stage('SonarQube Angular') {
      steps {
        withSonarQubeEnv(installationName: 'sonarqube-server', credentialsId: 'jenkins-sonarqube-token') {
          script {
            def scannerHome = tool 'sonarqube-scanner'  // Le nom que tu as défini dans Tools
            sh "${scannerHome}/bin/sonar-scanner"
          }
        }
      }
    }
  }
}
