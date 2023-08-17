pipeline {
    agent any

    environment {
        // Use Jenkins credentials binding
        CHKP_CLOUDGUARD_ID = credentials('chkp-cloudguard-id')
        CHKP_CLOUDGUARD_SECRET = credentials('chkp-cloudguard-secret')
        SHIFTLEFT_REGION = "eu1"
    }

    stages {
        stage('Clone Github repository') {
            steps {
                checkout scm
            }
        }

        stage('ShiftLeft Code Scan') {
            steps {
                script {
                    try {
                        sh './shiftleft code-scan -s .'
                    } catch (Exception e) {
                        echo "ShiftLeft code scan failed."
                    }
                }
            }
        }

        stage('webapp Docker image Build and scan prep') {
            steps {
                sh 'docker build -t dhouari/webapp .'
                sh 'docker save dhouari/webapp -o webapp.tar'
            }
        }

        stage('ShiftLeft Container Image Scan') {
            steps {
                script {
                    try {
                        sh './shiftleft image-scan -t 180 -i webapp.tar -r -2002 -e 4b89d765-1dfd-4c19-bf26-a34374142d42'
                    } catch (Exception e) {
                        echo "ShiftLeft docker image scan failed."
                    }
                }
            }
        }

        stage('Terraform config policy Scan') {
            steps {
                script {
                    try {
                        sh './shiftleft iac-assessment -l S3Bucket should have encryption.serverSideEncryptionRules -p ./terraform'
                    } catch (Exception e) {
                        echo "ShiftLeft Terraform config policy scan failed."
                    }
                }
            }
        }
    }
}

