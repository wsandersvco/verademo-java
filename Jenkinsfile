pipeline {
    agent {
        label "linux && java && amd64"
    }

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

            // post {
                // If Maven was able to run the tests, even if some of the test
                // failed, record the test results and archive the jar file.
                // success {
                    // withCredentials([usernamePassword(credentialsId: 'Veracode API Credentials', passwordVariable: 'vkey', usernameVariable: 'vid')]) {
                        // veracode applicationName: '$projectname', createProfile: true, criticality: 'VeryHigh', deleteIncompleteScanLevel: '0', fileNamePattern: '', replacementPattern: '', sandboxName: '', scanExcludesPattern: '', scanIncludesPattern: '', scanName: '', teams: 'Default Team', timeout: 60, uploadIncludesPattern: '**/**.jar,**/**.war', vid: vid, vkey: vkey, waitForScan: true
                    // }
                // }
            // }
        }
        stage('Veracode Pipeline Scan') {
            steps {
                // sh 'echo $PWD'
                // sh 'ls -lR'
                sh 'curl -O https://downloads.veracode.com/securityscan/pipeline-scan-LATEST.zip'
                sh 'unzip pipeline-scan-LATEST.zip pipeline-scan.jar'
                withCredentials([usernamePassword(credentialsId: 'Veracode API Credentials', passwordVariable: 'vkey', usernameVariable: 'vid')]) {
                    sh 'java -jar pipeline-scan.jar \
                      --veracode_api_id "${vid}" \
                      --veracode_api_key "${vkey}" \
                      --file "app/target/verademo.war" \
                      --fail_on_severity="Very High, High" \
                      --fail_on_cwe="80" \
                      --project_name "${JOB_NAME}" \
                      --project_url "${GIT_URL}" \
                      --project_ref "${GIT_COMMIT}"'
                }
            }
        }
    }
    post {
        always {
            archiveArtifacts artifacts: 'results.json', fingerprint: true
        }
    }
}
