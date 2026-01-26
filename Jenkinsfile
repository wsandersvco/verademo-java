pipeline {
    agent any

    tools {
        // Install the Maven version configured as "M3" and add it to the path.
        maven "M3"
        jdk "JDK25"
    }
    
    stages {
        stage("Check Version") {
            steps {
                sh 'echo JAVA_HOME is $JAVA_HOME'
                sh 'echo PATH is $PATH'
                sh 'java -version'
            }
        }
        stage('Build') {
            steps {
                // Get some code from a GitHub repository
                git branch: 'main', credentialsId: 'wsandersvco-all-repositories', url: 'https://github.com/wsandersvco/verademo-java'

                // Run Maven on a Unix agent.
                sh "mvn -Dmaven.test.failure.ignore=true -f app/pom.xml clean package"

                // To run Maven on a Windows agent, use
                // bat "mvn -Dmaven.test.failure.ignore=true clean package"
            }

            post {
                // If Maven was able to run the tests, even if some of the test
                // failed, record the test results and archive the jar file.
                success {
                    withCredentials([usernamePassword(credentialsId: 'Veracode API Credentials', passwordVariable: 'vkey', usernameVariable: 'vid')]) {
                        veracode applicationName: '$projectname', createProfile: true, criticality: 'VeryHigh', deleteIncompleteScanLevel: '0', fileNamePattern: '', replacementPattern: '', sandboxName: '', scanExcludesPattern: '', scanIncludesPattern: '', scanName: '', teams: 'Default Team', timeout: 60, uploadIncludesPattern: '**/**.jar,**/**.war', vid: vid, vkey: vkey, waitForScan: true
                    }
                }
            }
        }
    }
}
