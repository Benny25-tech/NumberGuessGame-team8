pipeline {
    agent any

    environment {
        TOMCAT_VERSION = "7.0.94"
        TOMCAT_HOME = "/home/ec2-user/apache-tomcat-${TOMCAT_VERSION}"
        WAR_DIRECTORY = "${TOMCAT_HOME}/webapps"
        TOMCAT_SERVER = "172.31.7.34"  // Private IP of your Tomcat server
        SSH_KEY = "Excel.pem"  // Your SSH key file
        SSH_USER = "ec2-user"  // EC2 user
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
                    stash name: 'warFile', includes: 'target/*.war'
                }
            }
        }

        stage('Setup and Deploy on Node 1 (Tomcat Server)') {
            agent { label 'node1' }
            steps {
                script {
                    sh '''
                    echo "Installing Java 17 on Node 1..."
                    sudo yum -y install java-17

                    echo "Downloading Tomcat ${TOMCAT_VERSION}..."
                    cd /tmp
                    wget -q https://archive.apache.org/dist/tomcat/tomcat-7/v${TOMCAT_VERSION}/bin/apache-tomcat-${TOMCAT_VERSION}.tar.gz

                    echo "Extracting Tomcat..."
                    sudo mkdir -p ${TOMCAT_HOME}
                    sudo tar xvf apache-tomcat-${TOMCAT_VERSION}.tar.gz -C ${TOMCAT_HOME} --strip-components=1
                    '''
                }
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                script {
                    unstash 'warFile'

                    sh '''
                    echo "Copying WAR file to Tomcat server..."
                    scp -i ${SSH_KEY} target/*.war ${SSH_USER}@${TOMCAT_SERVER}:${WAR_DIRECTORY}/

                    echo "Changing ownership of webapps directory..."
                    ssh -i ${SSH_KEY} ${SSH_USER}@${TOMCAT_SERVER} "sudo chown -R ec2-user:ec2-user ${WAR_DIRECTORY}"

                    echo "Restarting Tomcat..."
                    ssh -i ${SSH_KEY} ${SSH_USER}@${TOMCAT_SERVER} "sudo ${TOMCAT_HOME}/bin/shutdown.sh && sudo ${TOMCAT_HOME}/bin/startup.sh"
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
