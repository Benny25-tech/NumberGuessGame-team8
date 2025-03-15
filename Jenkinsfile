pipeline {
    agent any

    environment {
        TOMCAT_VERSION = "7.0.94"
        TOMCAT_HOME = "/home/ec2-user/apache-tomcat-${TOMCAT_VERSION}"
        WAR_DIRECTORY = "${TOMCAT_HOME}/webapps"
    }

    stages {
        stage('Checkout Code') {
            steps {
                git 'https://github.com/Benny25-tech/NumberGuessGame-team8.git'
            }
        }

        stage('Build with Maven') {
            steps {
                script {
                    // Run Maven build and ensure WAR file is created
                    sh 'mvn clean package'
                    sh 'ls target/*.war' // Ensure the WAR file is created and exists
                }
            }
        }

        stage('Setup and Deploy on Node 1') {
            agent { label 'node1' }
            steps {
                script {
                    sh '''
                    #!/bin/bash

                    echo "Installing Java 17 on Node 1..."
                    sudo yum -y install java-17

                    echo "Downloading Tomcat ${TOMCAT_VERSION}..."
                    cd /tmp
                    sudo wget -q https://archive.apache.org/dist/tomcat/tomcat-7/v${TOMCAT_VERSION}/bin/apache-tomcat-${TOMCAT_VERSION}.tar.gz

                    echo "Extracting Tomcat..."
                    sudo mkdir -p ${TOMCAT_HOME}
                    sudo tar xvf apache-tomcat-${TOMCAT_VERSION}.tar.gz -C ${TOMCAT_HOME} --strip-components=1

                    echo "Stopping any existing Tomcat instance..."
                    # Check if Tomcat is running and stop it, otherwise proceed
                    if pgrep -f "org.apache.catalina.startup.Bootstrap" > /dev/null; then
                        echo "Tomcat is running, stopping it..."
                        sudo ${TOMCAT_HOME}/bin/shutdown.sh || echo "Failed to stop Tomcat, proceeding with deployment."
                    else
                        echo "Tomcat is not running, proceeding with deployment."
                    fi

                    echo "Deploying WAR file..."
                    if [ -f "target/*.war" ]; then
                        sudo rm -rf ${TOMCAT_HOME}/webapps/*
                        sudo mv target/*.war ${WAR_DIRECTORY}/
                    else
                        echo "WAR file not found!"
                        exit 1
                    fi

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
                    # Stop Tomcat if it's running
                    sudo ${TOMCAT_HOME}/bin/shutdown.sh || echo "Tomcat is not running, proceeding with deployment."

                    # Move WAR file to Tomcat webapps directory
                    if [ -f "target/*.war" ]; then
                        sudo mv target/*.war ${WAR_DIRECTORY}/
                    else
                        echo "WAR file not found!"
                        exit 1
                    fi

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
