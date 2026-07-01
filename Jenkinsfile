pipeline {
    agent any
    tools {
        jdk 'jdk-11'
        maven 'maven-3.9.12'
    }
    options {
        buildDiscarder(logRotator(daysToKeepStr: '10', numToKeepStr: '10'))
    }
    parameters {
        string(name: 'JSLEE_XMPP_VERSION', defaultValue: '7.2.0', description: 'The major version for JAIN SLEE XMPP')
    }
    environment {
        XMPP_BUILD_VERSION = "${params.JSLEE_XMPP_VERSION}-${BUILD_NUMBER}"
    }
    stages {
        stage("Build") {
            steps {
                script {
                    XMPP_BUILD_VERSION = "${params.JSLEE_XMPP_VERSION}-${BUILD_NUMBER}"
                    currentBuild.displayName = "#${XMPP_BUILD_VERSION}"
                    currentBuild.description = "JAIN SLEE XMPP (${env.BRANCH_NAME})"
                }
                sh "mvn clean install -Dmaven.test.skip=true"
            }
        }
        stage('Set Version') {
            steps {
                sh "mvn versions:set -DgenerateBackupPoms=false -DnewVersion=${XMPP_BUILD_VERSION}"
                sh "mvn versions:commit"
            }
        }
        stage("Release") {
            steps {
                sh "mvn clean install -Prelease -Drelease.dir=../../../${XMPP_BUILD_VERSION} -Dmaven.test.skip=true"
            }
        }
        stage('Zip Resources') {
            steps {
                dir("${XMPP_BUILD_VERSION}/resources") {
                    sh "zip -r xmpp.zip xmpp"
                    sh 'rm -rf xmpp'
                }
            }
        }
        stage('Save Artifacts') {
            steps {
                archiveArtifacts artifacts: "${XMPP_BUILD_VERSION}/", followSymlinks: false, onlyIfSuccessful: true
            }
        }
        stage('Push to Repo') {
            when{anyOf {branch 'master'; branch 'release'}}
            steps {
                sh "mkdir -p /var/www/html/NAIKERI/jain-slee.xmpp/${XMPP_BUILD_VERSION}/"
                sh "cp -r ${XMPP_BUILD_VERSION}/ /var/www/html/NAIKERI/jain-slee.xmpp/${XMPP_BUILD_VERSION}/"
                sh "rm -rf ${XMPP_BUILD_VERSION}"
            }
        }
    }
    post {
        success { echo "JAIN-SLEE XMPP successfully built" }
        failure { echo "Building JAIN-SLEE XMPP failed" }
    }
}
