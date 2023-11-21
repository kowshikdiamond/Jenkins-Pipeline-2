pipeline {
    agent any

    environment {
        AWS_ACCESS_KEY_ID     = credentials('jenkins_aws_access_key_id')
        AWS_SECRET_ACCESS_KEY = credentials('jenkins_aws_secret_access_key')
        ANSIBLE_PRIVATE_KEY   = credentials('aws_private_key')
    }

    stages {
        stage('Install Dependencies') {
            steps {
                script {
                    // Install Python dependencies
                    sh 'pip install boto3 botocore'
                }
            }
        }

        stage('Create Server with Terraform') {
            steps {
                script {
                    // Change to the Terraform directory
                    dir('terraform') {
                        // Set AWS credentials as environment variables for Terraform
                        sh "export AWS_ACCESS_KEY_ID='${AWS_ACCESS_KEY_ID}'"
                        sh "export AWS_SECRET_ACCESS_KEY='${AWS_SECRET_ACCESS_KEY}'"

                        // Initialize Terraform
                        sh 'terraform init'

                        // Apply Terraform configuration
                        sh 'terraform apply --auto-approve'
                    }
                }
            }
        }

        stage('Copy SSH Key to Jenkins Server') {
            steps {
                script {
                    // Change to the Ansible directory
                    dir('ansible') {
                        // Copy SSH key to Jenkins server if it doesn't exist
                        withCredentials([sshUserPrivateKey(credentialsId: 'aws_private_key', keyFileVariable: 'keyfile', usernameVariable: 'user')]) {
                            // Get the current working directory
                            def currentDir = sh(script: 'pwd', returnStdout: true).trim()

                            // Check if the file exists before copying
                            if (!fileExists("${currentDir}/ssh-key.pem")) {
                                // Copy the key to the current working directory
                                sh "cp $KEYFILE ${currentDir}/ssh-key.pem"
                            }
                        }
                    }
                }
            }
        }

        stage('Configure Server with Ansible') {
            steps {
                script {
                    // Change to the Ansible directory
                    dir('ansible') {
                        // Run Ansible playbook with Docker credentials
                        withCredentials([usernamePassword(credentialsId: 'docker_credentials', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                            sh "ansible-playbook -e 'docker_username=${DOCKER_USERNAME}' -e 'docker_password=${DOCKER_PASSWORD}' ansible.yaml"
                        }
                    }
                }
            }
        }
    }
}

def fileExists(String filePath) {
    return sh(script: "[ -e ${filePath} ]", returnStatus: true) == 0
}