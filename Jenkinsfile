pipeline {
    agent any

    environment {
        TOMCAT_VERSION = "7.0.94"
        TOMCAT_HOME = "/home/ec2-user/apache-tomcat-${TOMCAT_VERSION}"
        WAR_NAME = "NumberGuessGame-2.0-SNAPSHOT.war"
        WAR_FILE = "target/${WAR_NAME}"
        DEPLOYMENT_SERVER = "172.31.7.34"
        SSH_CREDENTIALS = "Node1"
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
                    stash name: 'warFile', includes: WAR_FILE
                }
            }
        }

        stage('Install Java and Tomcat on Deployment Server') {
            steps {
                script {
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
                    unstash 'warFile'
                    
                    sshagent(credentials: [SSH_CREDENTIALS]) {
                        sh """
                        ssh -o StrictHostKeyChecking=no ec2-user@${DEPLOYMENT_SERVER} '
                            if [ -f ${TOMCAT_HOME}/webapps/${WAR_NAME} ]; then
                                echo "Removing old WAR file"
                                sudo rm -f ${TOMCAT_HOME}/webapps/${WAR_NAME}
                            fi
                        '
                        """
                    }
                    
                    sshagent(credentials: [SSH_CREDENTIALS]) {
                        sh """
                        scp -o StrictHostKeyChecking=no ${WORKSPACE}/${WAR_FILE} ec2-user@${DEPLOYMENT_SERVER}:${TOMCAT_HOME}/webapps/
                        """
                    }
                    
                    sshagent(credentials: [SSH_CREDENTIALS]) {
                        sh """
                        ssh -o StrictHostKeyChecking=no ec2-user@${DEPLOYMENT_SERVER} '
                            sudo ${TOMCAT_HOME}/bin/shutdown.sh
                            sudo ${TOMCAT_HOME}/bin/startup.sh
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
