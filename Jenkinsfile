pipeline {
    agent any

    environment {
        TOMCAT_VERSION = "7.0.94"
        TOMCAT_HOME = "/home/ec2-user/apache-tomcat-${TOMCAT_VERSION}"
        WAR_DIRECTORY = "${TOMCAT_HOME}/webapps"
        DEPLOYMENT_SERVER = "172.31.7.34"  // Deployment server's private IP address
        SSH_CREDENTIALS = "Node1"  // ID of the SSH credentials for the deployment server
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
                    // Run Maven build to ensure the WAR file is created
                    sh 'mvn clean package'
                    stash name: 'warFile', includes: '**/target/*.war'  // Stash the WAR file for later use
                }
            }
        }

        stage('Run Unit Tests') {
            steps {
                script {
                    // Run tests with Maven (JUnit tests)
                    sh 'mvn test'
                }
            }
        }

        stage('Install Java and Tomcat on Deployment Server') {
            steps {
                script {
                    // SSH into the deployment server to install Java and Tomcat (if needed)
                    sshagent(credentials: [SSH_CREDENTIALS]) {
                        sh """
                        ssh -o StrictHostKeyChecking=no ec2-user@${DEPLOYMENT_SERVER} '
                            sudo yum -y install java-17
                            cd /tmp
                            sudo wget https://archive.apache.org/dist/tomcat/tomcat-7/v${TOMCAT_VERSION}/bin/apache-tomcat-${TOMCAT_VERSION}.tar.gz
                            sudo tar xvf apache-tomcat-${TOMCAT_VERSION}.tar.gz -C /home/ec2-user/
                            sudo chown -R ec2-user:ec2-user /home/ec2-user/apache-tomcat-${TOMCAT_VERSION}
                            sudo /home/ec2-user/apache-tomcat-${TOMCAT_VERSION}/bin/startup.sh
                        '
                        """
                    }
                }
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                script {
                    unstash 'warFile'  // Retrieve the WAR file from the stash

                    // Deploy WAR file to Tomcat's webapps directory
                    sshagent(credentials: [SSH_CREDENTIALS]) {
                        sh """
                        ssh -o StrictHostKeyChecking=no ec2-user@${DEPLOYMENT_SERVER} '
                            sudo /home/ec2-user/apache-tomcat-${TOMCAT_VERSION}/bin/shutdown.sh
                            mv target/NumberGuessGame-1.0-SNAPSHOT.war /home/ec2-user/apache-tomcat-${TOMCAT_VERSION}/webapps/
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
