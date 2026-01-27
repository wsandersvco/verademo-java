pipeline {
    agent {
        label 'linux && java && amd64'
    }

    environment {
        VERACODE_APP_NAME = 'Jenkins Pipeline - verademo-java'      // App Name in the Veracode Platform
    }

    tools {
        // Install the Maven version configured as "M3" and add it to the path.
        maven 'M3'
        jdk 'JDK25'
    }

    stages {
        stage('Check Version') {
            steps {
                sh 'echo JAVA_HOME is $JAVA_HOME'
                sh 'echo PATH is $PATH'
                sh 'java -version'
            }
        }
        stage('Build') {
            steps {
                // Get some code from a GitHub repository
                git branch: 'main',
                    credentialsId: 'wsandersvco-all-repositories',
                    url: 'https://github.com/wsandersvco/verademo-java'

                // Run Maven on a Unix agent.
                sh 'mvn -Dmaven.test.failure.ignore=true -f app/pom.xml clean package'

            // To run Maven on a Windows agent, use
            // bat "mvn -Dmaven.test.failure.ignore=true clean package"
            }
        }
        stage('Veracode Upload and Scan') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'Veracode API Credentials',
                    passwordVariable: 'vkey', usernameVariable: 'vid')]) {
                    veracode applicationName: '$VERACODE_APP_NAME', canFailJob: true, createProfile: true,
                        criticality: 'VeryHigh', deleteIncompleteScanLevel: '1', fileNamePattern: 'verademo.war',
                        replacementPattern: 'verademo.war', sandboxName: '', scanExcludesPattern: '',
                        scanIncludesPattern: '', scanName: '$buildnumber', teams: 'Default Team',
                        timeout: 60, uploadIncludesPattern: 'app/target/verademo.war',
                        vid: vid, vkey: vkey, waitForScan: true
                }
            }
        }
    }
}
