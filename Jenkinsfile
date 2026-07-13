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
                bat 'npmw.cmd run ci:frontend:test'
            }
        }

        stage('Backend Tests') {
            steps {
                bat 'npmw.cmd run ci:backend:test'
            }
        }

        stage('Build JAR') {
            steps {
                bat 'npmw.cmd run java:jar:prod'
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
