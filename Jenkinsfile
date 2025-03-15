pipeline {
    agent any

    environment {
        TOMCAT_HOME = "/home/ec2-user/apache-tomcat-7"
        WAR_FILE = "target/NumberGuessGame.war"
    }

    stages {
        stage('Checkout Code') {
            steps {
                git 'https://github.com/Benny25-tech/NumberGuessGame-team8.git'
            }
        }

        stage('Build with Maven') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Setup and Deploy on Node 1') {
            agent { label 'node1' } // Runs only on Node 1
            steps {
                script {
                    sh '''
                    #!/bin/bash
                    echo "Installing Java 17 on Node 1..."
                    sudo yum -y install java-17

                    echo "Downloading Tomcat 7.0.94..."
                    cd /tmp
                    sudo wget -q https://archive.apache.org/dist/tomcat/tomcat-7/v7.0.94/bin/apache-tomcat-7.0.94.tar.gz

                    echo "Extracting Tomcat..."
                    sudo mkdir -p ${TOMCAT_HOME}
                    sudo tar xvf apache-tomcat-7.0.94.tar.gz -C ${TOMCAT_HOME} --strip-components=1

                    echo "Stopping any existing Tomcat instance..."
                    sudo ${TOMCAT_HOME}/bin/shutdown.sh || true
                    
                    echo "Deploying WAR file..."
                    sudo rm -rf ${TOMCAT_HOME}/webapps/ROOT*
                    sudo cp ${WAR_FILE} ${TOMCAT_HOME}/webapps/ROOT.war

                    echo "Starting Tomcat..."
                    sudo ${TOMCAT_HOME}/bin/startup.sh
                    '''
                }
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                script {
                    sh '''
                    # Stop Tomcat
                    sudo ${TOMCAT_HOME}/bin/shutdown.sh || true
                    
                    # Copy WAR file to Tomcat webapps directory
                    sudo cp ${WAR_FILE} ${TOMCAT_HOME}/webapps/ROOT.war

                    # Start Tomcat again
                    sudo ${TOMCAT_HOME}/bin/startup.sh
                    '''
                }
            }
        }
    }

    post {
        success {
            script {
                mail to: "ruthnwaigwe33@gmail.com",
                    subject: "Build and Deployment Successful - ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                    body: "Congratulations! The build and deployment were successful.\n\nCheck console output at ${env.BUILD_URL}"
            }
        }
        failure {
            script {
                mail to: "ruthnwaigwe33@gmail.com",
                    subject: "Build and Deployment Failed - ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                    body: "Oops! The build and deployment failed.\n\nCheck console output at ${env.BUILD_URL}"
            }
        }
    }
}
