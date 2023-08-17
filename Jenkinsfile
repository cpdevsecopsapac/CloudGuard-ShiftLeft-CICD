pipeline {
    agent any

    environment {
        // Use Jenkins credentials binding
        CHKP_CLOUDGUARD_ID = credentials('chkp-cloudguard-id')
        CHKP_CLOUDGUARD_SECRET = credentials('chkp-cloudguard-secret')
        SHIFTLEFT_REGION = "eu1"
        SPECTRAL_DSN = credentials('spectral-dsn')
    }

    stages {
        stage('Clone Github repository') {
            steps {
                checkout scm
            }
        }

        stage('install Spectral') {
            steps {
                sh "curl -L 'https://spectral-eu.checkpoint.com/latest/x/sh?dsn=$SPECTRAL_DSN' | sh"
            }
        }

        stage('ShiftLeft Code Scan') {
            steps {
                script {
                    try {
                        sh 'chmod +x shiftleft'
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

        stage('ShiftLeft Container Image Scan Online') {
            steps {
                script {
                    try {
                        sh 'chmod +x shiftleft'
                        sh './shiftleft image-scan -t 180 -i webapp.tar -r -2002 -e 4b89d765-1dfd-4c19-bf26-a34374142d42'
                    } catch (Exception e) {
                        echo "ShiftLeft docker image scan failed."
                    }
                }
            }
        }

        stage('Shiftleft Container Image Scan Offline') {
            steps {
                script {
                    try {
                        sh 'chmod +x shiftleft'
                        sh './shiftleft image-scan -t 180 -i webapp.tar'
                    } catch (Exception e) {
                        echo "ShiftLeft docker Image scan failed."
                    }
                }
            }
        }
    }
}
