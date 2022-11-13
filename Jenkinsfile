pipeline{
   agent any
   options {
           timeout(time: 10, unit: 'MINUTES')
   }
   stages{
        stage('Validate project') {
            steps{
                sh 'mvn validate'
            }
        }
        stage('Maven build') {
            steps{
                sh 'mvn clean install'
            }
        }
        stage('Unit testing') {
            steps{
                sh 'mvn test'
            }
        }
        stage('Run Sonarqube') {
            environment{
                scannerHome = tool 'ibt-sonarqube';
                  }
                steps{
                    withSonarQubeEnv(credentialsId: 'SQ-student', installationName: 'IBT sonarqube') {
                        sh "${scannerHome}/bin/sonar-scanner"
                  }
            }
        }
        stage('Run Dependency check') {
            steps{
                dependencyCheck additionalArguments: '''
                           -o "./"
                           -s "./"
                           -f "ALL"
                           --prettyPrint''', odcInstallation: 'dependency-check'
                dependencyCheckPublisher pattern: 'dependency-check-report.xml'
            }
        }
        stage('Push to Artifactory (Jfrog)') {
            steps{
                configFileProvider([configFile(fileId: '5d0920bc-97c5-4877-8aa4-2f61975fa9fc', variable: 'MAVEN_SETTINGS_XML')]) {
                      echo 'mvn -U --batch-mode -s $MAVEN_SETTINGS_XML clean deploy'
                }
            }
        }

        stage('Configure our VM(s)') {
           steps{
               catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                        ansiblePlaybook(
                            credentialsId: 'ooghenekaro-ssh',
                             installation: 'ansible',
                             inventory: 'ansible/hosts',
                             playbook: 'ansible/tomcat.yaml'
                        )
               }
           }
       }
        stage('Deploy to DEV') {
            steps {
                ansiblePlaybook(
                    playbook: 'ansible/deploy-war.yaml',
                    inventory: 'ansible/hosts',
                    credentialsId: 'ooghenekaro-ssh',
                    colorized: true,
                    extraVars: [
                        "myHosts" : "devServer",
                        "artifact": "${WORKSPACE}/target/hello-maven-1.0-SNAPSHOT.war"
                    ]
                )
            }
        }
        stage('Approval to Deploy to PROD') {
            steps{
                input message: 'Ready to Deploy to Prod',
                      submitter: 'ooghenekaro'
            }
        }
        stage('Deploy to PROD') {
            steps {
                ansiblePlaybook(
                    playbook: 'ansible/deploy-war.yaml',
                    inventory: 'ansible/hosts',
                    credentialsId: 'ooghenekaro-ssh',
                    colorized: true,
                    extraVars: [
                        "myHosts" : "prodServer",
                        "artifact": "${WORKSPACE}/target/hello-maven-1.0-SNAPSHOT.war"
                    ]
                )
            }
        }
   }
}