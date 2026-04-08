pipeline {
    agent any

    tools {
        nodejs 'node18'
        jdk 'jdk17'
    }

    environment {
        AWS_DEFAULT_REGION = 'ap-south-1'
        S3_BUCKET = 'devopssampleapp'
        APP_NAME = 'FrontendSampleApp'
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/TejasG30/ForntendSampleApp.git'
            }
        }

        stage('Validate') {
            steps {
                sh 'ls -l'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh 'sonar-scanner'
                }
            }
        }

        //  BUILD STAGE (ADDED)
        stage('Build Angular App') {
            steps {
                sh 'npm run build -- --configuration production'
            }
        }

        //  Terraform - Create S3
        stage('Terraform Init') {
            steps {
                sh 'terraform -chdir=terraform/site init'
            }
        }

        stage('Terraform Apply') {
            steps {
                withAWS(credentials: 'aws-creds', region: 'ap-south-1') {
                    sh 'terraform -chdir=terraform/site apply -auto-approve'
                }
            }
        }

        //  Deploy Build to S3
        stage('Deploy to S3') {
            steps {
                withAWS(credentials: 'aws-creds', region: 'ap-south-1') {
                    sh '''
                    aws s3 sync dist/$APP_NAME/ s3://$S3_BUCKET --delete
                    '''
                }
            }
        }
    }
}
