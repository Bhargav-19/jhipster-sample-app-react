#!/usr/bin/env groovy

pipeline {
    agent any

    environment {
        // Jenkins on Windows often runs as SYSTEM; redirect caches to the workspace
        HOME = "${WORKSPACE}"
        USERPROFILE = "${WORKSPACE}"
        TEMP = "${WORKSPACE}\\.tmp"
        TMP = "${WORKSPACE}\\.tmp"
        MAVEN_USER_HOME = "${WORKSPACE}\\.m2"
        NPM_CONFIG_CACHE = "${WORKSPACE}\\.npm"
        // Puppeteer (from lighthouse/cypress-audit) cannot download Chromium on restricted networks
        PUPPETEER_SKIP_DOWNLOAD = 'true'
        // Must match the SonarQube server name in Jenkins → Manage Jenkins → System → SonarQube servers
        SONARQUBE_SERVER = 'SonarQube'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
                bat 'if not exist ".tmp" mkdir ".tmp"'
            }
        }

        stage('Check Java') {
            steps {
                bat 'java -version'
            }
        }

        stage('Clean') {
            steps {
                bat 'mvnw.cmd -ntp clean'
            }
        }

        stage('Install Tools') {
            steps {
                bat 'mvnw.cmd -ntp -Pwebapp frontend:install-node-and-npm@install-node-and-npm'
            }
        }

        stage('Install Dependencies') {
            steps {
                bat 'npmw.cmd install'
            }
        }

        stage('Frontend Tests') {
            steps {
                // Use explicit prod script; npm $npm_package_config_* vars are not expanded on Windows cmd
                bat 'npmw.cmd run webapp:build:prod && npmw.cmd run test'
            }
        }

        stage('Backend Tests') {
            steps {
                bat 'mvnw.cmd --version'
                bat 'mvnw.cmd -ntp javadoc:javadoc --batch-mode'
                bat 'mvnw.cmd -ntp checkstyle:check --batch-mode'
                bat 'mvnw.cmd -ntp -Dskip.installnodenpm -Dskip.npm verify --batch-mode -Dlogging.level.ROOT=OFF -Dlogging.level.tech.jhipster=OFF -Dlogging.level.io.github.jhipster.sample=OFF -Dlogging.level.org.springframework=OFF -Dlogging.level.org.springframework.web=OFF -Dlogging.level.org.springframework.security=OFF -Pprod'
            }
        }

        stage('Build JAR') {
            steps {
                bat 'mvnw.cmd -ntp verify -DskipTests --batch-mode -Pprod'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv("${SONARQUBE_SERVER}") {
                    bat 'mvnw.cmd -ntp initialize sonar:sonar'
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
    }

    post {
        always {
            junit allowEmptyResults: true, testResults: 'target/surefire-reports/TEST-*.xml,target/failsafe-reports/TEST-*.xml,target/test-results/**/TESTS-*.xml'
            archiveArtifacts artifacts: 'target/*.jar', allowEmptyArchive: true, fingerprint: true
            cleanWs()
        }
    }
}
