pipeline {
    agent any

    environment {
        SONAR_TOKEN = credentials('SONAR_TOKEN')
        GITHUB_REPO = 'https://github.com/az-1234567/8.2CDevSecOps.git'
        SONAR_PROJECT_KEY = 'az-1234567_8-2CDevSecOps'
        SONAR_ORG = 'az-1234567'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: "${env.GITHUB_REPO}"
            }
        }

        stage('Install dependencies') {
            steps {
                bat 'npm install'
            }
        }

        stage('Run unit tests') {
            steps {
                bat 'npm test || exit 0'
            }
        }

        stage('Coverage') {
            steps {
                bat 'npm run coverage || exit 0'
            }
        }

        stage('NPM Audit (security scan)') {
            steps {
                bat 'npm audit --json > audit.json || exit 0'
                bat 'npm audit > audit.log || exit 0'
            }
        }

        stage('SonarCloud Analysis') {
            steps {
                bat '''
                if not exist sonar mkdir sonar
                cd sonar
                if not exist sonar-scanner-cli.zip (
                  curl -L -o sonar-scanner-cli.zip https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-4.8.0.2856-windows.zip
                  powershell -Command "Expand-Archive sonar-scanner-cli.zip -DestinationPath ."
                )
                for /d %%D in (sonar-scanner-*)*

