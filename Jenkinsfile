pipeline {
    agent any

	tools {
	    jdk 'JDK 11'
		maven 'Maven_3.8.5'
	}
	parameters { string(name: 'JAIN_SLEE_XMPP_MAJOR_VERSION', defaultValue: '7.2.0', description: 'The major version for JAIN SLEE XMPP version') }

    environment{
        XMPP_BUILD_VERSION = "${params.JAIN_SLEE_XMPP_MAJOR_VERSION}-${BUILD_NUMBER}"
    }

	stages{
		stage("Build ") {
			steps {
			    sh 'java -version'
				script {
				    XMPP_BUILD_VERSION = "${params.JAIN_SLEE_XMPP_MAJOR_VERSION}-${BUILD_NUMBER}"
                    currentBuild.displayName = "#${XMPP_BUILD_VERSION}"
                    currentBuild.description = "JAIN SLEE XMPP (${env.BRANCH_NAME})"
                }
				echo "Building JAIN SLEE XMPP version (#${XMPP_BUILD_VERSION})"
                sh "mvn versions:set -DgenerateBackupPoms=false -DnewVersion=${XMPP_BUILD_VERSION} clean install -DskipTests"
				sh 'mvn clean install -DskipTests'
                echo "Maven build completed."
            }
        }

		stage("Release") {
			steps {
				echo "Building a wildfly release version of #${XMPP_BUILD_VERSION}"
                sh 'find . -type d -name \\"target\\" -exec r -r {} +'
                sh "mvn versions:set -DnewVersion=${XMPP_BUILD_VERSION} clean install -Prelease -Drelease.dir=../../../${XMPP_BUILD_VERSION} -Dmaven.test.skip=true"
                echo "Building wildfly version completed."
            }
        }
	}
}
