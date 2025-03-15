pipeline {
    agent any  // Runs checkout and build on Jenkins agent

    environment {
        TOMCAT_VERSION = "9.0.21"
        TOMCAT_HOME = "/opt/tomcat"
        WAR_FILE = "/home/ec2-user/NumberGuessGame-team8/target/NumberGuessGame.war"
        REPO_DIR = "/home/ec2-user/NumberGuessGame-team8"
    }

    stages {
        stage('Checkout Code') {
            steps {
                script {
                    echo "Cloning repository..."
                    if ([ -d "${REPO_DIR}" ]) {
                        cd ${REPO_DIR}
                        sh 'git pull origin master'
                    } else {
                        sh "git clone https://github.com/Benny25-tech/NumberGuessGame-team8.git ${REPO_DIR}"
                    }
                }
            }
        }

        stage('Build with Maven') {
            steps {
                script {
                    sh '''
                    cd ${REPO_DIR}
                    mvn clean package
                    '''
                }
            }
        }

        stage('Setup and Deploy on Node 1') {
            agent { label 'node1' } // Runs only on Node 1
            steps {
                script {
                    sh '''
                    #!/bin/bash
                    echo "Installing Java 17 on Node 1..."
                    sudo yum -y install java-17-openjdk

                    echo "Downloading Tomcat..."
                    cd /tmp
                    sudo wget -q https://archive.apache.org/dist/tomcat/tomcat-9/v${TOMCAT_VERSION}/bin/apache-tomcat-${TOMCAT_VERSION}.tar.gz  

                    echo "Extracting Tomcat..."
                    sudo mkdir -p ${TOMCAT_HOME}
                    sudo tar -xvzf apache-tomcat-${TOMCAT_VERSION}.tar.gz -C ${TOMCAT_HOME} --strip-components=1
                    sudo chmod +x ${TOMCAT_HOME}/bin/*.sh

                    echo "Stopping any existing Tomcat instance..."
                    sudo ${TOMCAT_HOME}/bin/shutdown.sh || true
                    
                    echo "Deploying WAR file..."
                    sudo rm -rf ${TOMCAT_HOME}/webapps/ROOT*
                    sudo cp ${WAR_FILE} ${TOMCAT_HOME}/webapps/ROOT.war

                    echo "Restarting Tomcat..."
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
