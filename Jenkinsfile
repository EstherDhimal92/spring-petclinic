pipeline {
  agent any

  // Every 5 minutes on Mondays (DOW 1 = Monday)
  triggers {
    cron('H/5 * * * 1')
  }

  options { timestamps() }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Build + Test') {
      steps {
        sh 'chmod +x mvnw'
        sh './mvnw -B clean test'
      }
      post {
        always {
          junit 'target/surefire-reports/*.xml'
        }
      }
    }

    stage('JaCoCo Coverage Report') {
      steps {
        // JaCoCo plugin already exists in pom.xml, this will generate target/site/jacoco
        sh './mvnw -B jacoco:report'
      }
      post {
        always {
          // Publish the HTML report in Jenkins (needs "HTML Publisher" plugin)
          publishHTML(target: [
            allowMissing: false,
            alwaysLinkToLastBuild: true,
            keepAll: true,
            reportDir: 'target/site/jacoco',
            reportFiles: 'index.html',
            reportName: 'JaCoCo Coverage Report'
          ])

          // Archive the report folder
          archiveArtifacts artifacts: 'target/site/jacoco/**', fingerprint: true
        }
      }
    }

    stage('Package Artifact') {
      steps {
        sh './mvnw -B -DskipTests package'
      }
      post {
        success {
          archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
        }
      }
    }
  }
}
