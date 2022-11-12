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
            steps{
                withSonarQubeEnv(credentialsId: 'Sonar-key', installationName: 'SonarServer') {
                         sh "mvn clean package sonar:sonar"
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
                configFileProvider([configFile(fileId: '9cd879f3-0d67-4b90-b1d3-82fb6a73442b', variable: 'MAVEN_SETTINGS_XML')]) {
                      sh 'mvn -U --batch-mode -s $MAVEN_SETTINGS_XML clean deploy'
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