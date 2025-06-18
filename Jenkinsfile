pipeline {
  agent { label 'agent-node' }

  tools {
    jdk 'Java 17'
    maven 'Maven'
    
  }
   environment {
    MAVEN_OPTS = '-Xmx1024m'
    DOCKER_HUB_USER = 'rima603'
    DOCKER_HUB_PASSWORD = credentials('dockerhub')
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
          dir('ebanking-frontend') {
        withSonarQubeEnv(installationName: 'sonarqube-server', credentialsId: 'jenkins-sonarqube-token') {
          script {
            def scannerHome = tool 'sonarqube-scanner'  
            sh "${scannerHome}/bin/sonar-scanner"
          }
        }
      }
    }
  }
      stage("Build Docker Images") {
      steps {
        script {
          dir('ebanking-backend') {
            sh """
              docker build -t rima603/bankabackend:${BUILD_NUMBER} .
              docker tag rima603/bankabackend:${BUILD_NUMBER} rima603/bankabackend:latest
            """
          }
          dir('ebanking-frontend') {
            sh """
              docker build -t rima603/bankafront:${BUILD_NUMBER} .
              docker tag rima603/bankafront:${BUILD_NUMBER} rima603/bankafront:latest
            """
          }
        }
      }
    }
stage("Trivy Security Scan") {
      steps {
        script {
          sh '''
            echo "🔍 Trivy scan rapide sur les images Docker..."

            mkdir -p $HOME/.cache/trivy

            docker run --rm \
              -v /var/run/docker.sock:/var/run/docker.sock \
              -v $HOME/.cache/trivy:/root/.cache/ \
              aquasec/trivy image \
              --scanners vuln \
             --skip-db-update \
              --timeout 10m \
              --severity HIGH,CRITICAL \
              rima603/bankabackend:${BUILD_NUMBER} || true

            docker run --rm \
              -v /var/run/docker.sock:/var/run/docker.sock \
              -v $HOME/.cache/trivy:/root/.cache/ \
              aquasec/trivy image \
              --scanners vuln \
              --skip-db-update \
              --timeout 2m \
              --severity HIGH,CRITICAL \
              rima603/bankafront:${BUILD_NUMBER} || true
          '''
        }
      }
    }
    
      stage("Push Docker Images to Docker Hub") {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
          sh """
            echo "${DOCKER_PASS}" | docker login -u "${DOCKER_USER}" --password-stdin
            docker push rima603/bankabackend:${BUILD_NUMBER}
            docker push rima603/bankabackend:latest
            docker push rima603/bankafront:${BUILD_NUMBER}
            docker push rima603/bankafront:latest
          """
        }
      }
    }
}
}
