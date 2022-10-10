pipeline {
    agent any 
    stages {  
        stage('Vadidate mavn project') { 
            steps {
                sh "mvn validate"
            }
        }
        stage('Run maven test') { 
            steps {
                sh "mvn test"
            }
        }
        stage('Run clean install') { 
            steps {
                echo "mvn clean insall"
            }
        }
        
    }
}
