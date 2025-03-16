pipeline {
    agent any

    environment {
        TOMCAT_VERSION = "7.0.94"
        TOMCAT_HOME = "/home/ec2-user/apache-tomcat-${TOMCAT_VERSION}"
        WAR_FILE = "target/NumberGuessGame-1.0-SNAPSHOT.war"  // WAR file built on Jenkins server
        DEPLOYMENT_SERVER = "172.31.7.34"  // Tomcat (Deployment) server's private IP
        SSH_CREDENTIALS = "Node1"  // SSH credentials ID for the deployment server
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
                    sh 'mvn clean package'
                    stash name: 'warFile', includes: WAR_FILE  // Stash WAR file
                }
            }
        }

        stage('Transfer WAR File to Tomcat Server') {
            steps {
                script {
                    unstash 'warFile'  // Retrieve the WAR file from the stash
                    
                    sshagent(credentials: [SSH_CREDENTIALS]) {
                        sh """
                        scp -o StrictHostKeyChecking=no ${WAR_FILE} ec2-user@${DEPLOYMENT_SERVER}:/home/ec2-user/
                        """
                    }
                }
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                script {
                    sshagent(credentials: [SSH_CREDENTIALS]) {
                        sh """
                        ssh -o StrictHostKeyChecking=no ec2-user@${DEPLOYMENT_SERVER} '
                            sudo /home/ec2-user/apache-tomcat-${TOMCAT_VERSION}/bin/shutdown.sh
                            
                            # Remove old WAR file if it exists
                            if [ -f /home/ec2-user/apache-tomcat-${TOMCAT_VERSION}/webapps/NumberGuessGame-1.0-SNAPSHOT.war ]; then
                                echo "Removing old WAR file"
                                sudo rm -f /home/ec2-user/apache-tomcat-${TOMCAT_VERSION}/webapps/NumberGuessGame-1.0-SNAPSHOT.war
                            fi

                            # Move the new WAR file into Tomcat's webapps directory
                            sudo mv /home/ec2-user/NumberGuessGame-1.0-SNAPSHOT.war /home/ec2-user/apache-tomcat-${TOMCAT_VERSION}/webapps/

                            # Start Tomcat server
                            sudo /home/ec2-user/apache-tomcat-${TOMCAT_VERSION}/bin/startup.sh
                        '
                        """
                    }
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
