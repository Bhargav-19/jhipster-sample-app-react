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
        DEPLOY_DIR = "${WORKSPACE}\\deploy"
        // Puppeteer (from lighthouse/cypress-audit) cannot download Chromium on restricted networks
        PUPPETEER_SKIP_DOWNLOAD = 'true'
        // Must match the SonarQube server name in Jenkins → Manage Jenkins → System → SonarQube servers
        SONARQUBE_SERVER = 'SonarQube'
    }

    stages {
        stage('Checkout') {
            steps {
                echo '--- Checkout: pull latest code from Git ---'
                checkout scm
                bat 'if not exist ".tmp" mkdir ".tmp"'
                bat 'if not exist "deploy" mkdir "deploy"'
            }
        }

        stage('Quality Check') {
            steps {
                echo '--- Quality Check: tests, code style, SonarQube, and quality gate ---'
                bat 'mvnw.cmd -ntp -Pwebapp frontend:install-node-and-npm@install-node-and-npm'
                bat 'npmw.cmd install'
                bat 'npmw.cmd run webapp:build:prod && npmw.cmd run test'
                bat 'mvnw.cmd -ntp checkstyle:check --batch-mode'
                bat 'mvnw.cmd -ntp -Dskip.installnodenpm -Dskip.npm verify --batch-mode -Dlogging.level.ROOT=OFF -Dlogging.level.tech.jhipster=OFF -Dlogging.level.io.github.jhipster.sample=OFF -Dlogging.level.org.springframework=OFF -Dlogging.level.org.springframework.web=OFF -Dlogging.level.org.springframework.security=OFF -Pprod'
                withSonarQubeEnv("${SONARQUBE_SERVER}") {
                    bat 'mvnw.cmd -ntp initialize sonar:sonar'
                }
                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Build') {
            steps {
                echo '--- Build: create production JAR ---'
                bat 'mvnw.cmd -ntp verify -DskipTests --batch-mode -Pprod'
            }
        }

        stage('Deploy') {
            steps {
                echo '--- Deploy: copy JAR to deploy folder ---'
                bat 'copy /Y target\\*.jar %DEPLOY_DIR%\\'
                echo "Application JAR copied to ${DEPLOY_DIR}"
            }
        }
    }

    post {
        always {
            junit allowEmptyResults: true, testResults: 'target/surefire-reports/TEST-*.xml,target/failsafe-reports/TEST-*.xml,target/test-results/**/TESTS-*.xml'
            archiveArtifacts artifacts: 'target/*.jar,deploy/*.jar', allowEmptyArchive: true, fingerprint: true
            cleanWs()
        }
    }
}
