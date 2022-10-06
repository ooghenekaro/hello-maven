pipeline {
    agent any 
    stages {
        stage('Git checkout') { 
            steps {
                echo "checkout Complete"
            }
        }
        stage('Sonarqube test') { 
            steps {
                echo "Test Complete"
            }
        }
        stage('Build code using Maven') { 
            steps {
                echo "Maven Build Complete"
            }
        }
        stage('Run Unit Test') { 
            steps {
                echo "Unit Test Complete"
            }
        }
        stage('Push package to Registry') { 
            steps {
                echo "Code Pushed to Registry"
            }
        }
        stage('Perform Dynamic code analysis') { 
            steps {
                echo "Perform Dynamic code analysis"
            }
        }
    }
}
