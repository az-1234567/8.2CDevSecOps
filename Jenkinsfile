pipeline {
  agent {
    docker { image 'node:18' }
  }

  environment {
    SONAR_TOKEN = credentials('SONAR_TOKEN')
    GITHUB_REPO = 'https://github.com/az-1234567/8.2CDevSecOps.git'
    SONAR_PROJECT_KEY = 'YOUR_SONAR_PROJECT_KEY'
    SONAR_ORG = 'YOUR_SONAR_ORG'
  }

  stages {
    stage('Checkout') {
      steps {
        git branch: 'main', url: "${env.GITHUB_REPO}"
      }
    }

    stage('Install dependencies') {
      steps {
        sh 'npm ci 2>&1 | tee install.log'
      }
    }

    stage('Run unit tests') {
      steps {
        sh 'npm test 2>&1 | tee test.log || true'
      }
    }

    stage('Coverage') {
      steps {
        sh 'npm run coverage 2>&1 | tee coverage.log || true'
      }
    }

    stage('NPM Audit (security scan)') {
      steps {
        sh 'npm audit --json > audit.json || true'
        sh 'cat audit.json | jq . > audit_pretty.json || true || true'
        sh 'cat audit_pretty.json 2>/dev/null || true'
        sh 'npm audit 2>&1 | tee audit.log || true'
      }
    }

    stage('SonarCloud Analysis') {
      steps {
        sh '''
          mkdir -p sonar
          cd sonar
          if [ ! -f sonar-scanner-cli.zip ]; then
            curl -L -o sonar-scanner-cli.zip https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-4.8.0.2856-linux.zip
            unzip sonar-scanner-cli.zip
          fi
          cd sonar-scanner-*/
          ./bin/sonar-scanner \
            -Dsonar.login=${SONAR_TOKEN} \
            -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
            -Dsonar.organization=${SONAR_ORG} \
            -Dsonar.host.url=https://sonarcloud.io \
            -Dsonar.sources=../
        '''
      }
    }

    stage('Archive logs & coverage') {
      steps {
        archiveArtifacts artifacts: 'install.log,test.log,coverage.log,audit.log,audit.json,**/coverage/lcov.info', allowEmptyArchive: true
      }
    }
  }

  post {
    always {
      sh 'tail -n 4000 $(echo ${WORKSPACE}/install.log || true) > console_snapshot.log || true'
      archiveArtifacts artifacts: 'console_snapshot.log', allowEmptyArchive: true
    }
  }
}
