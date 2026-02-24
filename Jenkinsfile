pipeline {
  agent any

  triggers {
    // Every minute on Tuesdays (used for today: 2026-02-24)
    cron('* * * * 2')
  }

  options {
    timestamps()
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build and Test') {
      steps {
        script {
          if (isUnix()) {
            sh './mvnw -B clean test'
          } else {
            bat 'mvnw.cmd -B clean test'
          }
        }
      }
    }

    stage('JaCoCo Coverage Report') {
      steps {
        script {
          if (isUnix()) {
            sh './mvnw -B jacoco:report'
          } else {
            bat 'mvnw.cmd -B jacoco:report'
          }
        }
      }
      post {
        always {
          archiveArtifacts artifacts: 'target/site/jacoco/**', fingerprint: true, allowEmptyArchive: true
          script {
            try {
              recordCoverage tools: [[parser: 'JACOCO', pattern: 'target/site/jacoco/jacoco.xml']]
            } catch (Exception e) {
              echo 'Coverage plugin is not installed or recordCoverage is unavailable. JaCoCo HTML is still archived in artifacts.'
            }
          }
        }
      }
    }

    stage('Package Artifact') {
      steps {
        script {
          if (isUnix()) {
            sh './mvnw -B -DskipTests package'
          } else {
            bat 'mvnw.cmd -B -DskipTests package'
          }
        }
      }
    }
  }

  post {
    always {
      archiveArtifacts artifacts: 'target/*.jar', fingerprint: true, allowEmptyArchive: true
      junit testResults: 'target/surefire-reports/*.xml', allowEmptyResults: true
    }
  }
}
