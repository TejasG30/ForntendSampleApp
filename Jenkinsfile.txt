pipeline {
    agent any

    environment {
        AWS_DEFAULT_REGION = "us-east-1"
        S3_BUCKET = "sans-devops-static-site"
    }

    stages {

        stage('Validate') {
            steps {
                sh 'ls -l src'
            }
        }

        stage('Test - SonarQube') {
            steps {
                script {
                    def scannerHome = tool 'sonar'
                    withSonarQubeEnv('sonar') {
                        sh """
                          ${scannerHome}/bin/sonar-scanner \
                          -Dsonar.projectKey=devops-static-site \
                          -Dsonar.projectName=DevOps-Static-Site \
                          -Dsonar.sources=src
                        """
                    }
                }
            }
        }

        stage('Terraform Init') {
            steps {
                sh 'terraform -chdir=terraform/site init'
            }
        }

        stage('Terraform Apply') {
            steps {
                sh 'terraform -chdir=terraform/site apply -auto-approve'
            }
        }

        stage('Deploy to S3') {
            steps {
                sh 'aws s3 sync src/ s3://$S3_BUCKET --delete'
            }
        }

        /* 🔥 NEW STAGE – DO NOT MODIFY ANYTHING ABOVE */
        stage('Invalidate CloudFront Cache') {
            steps {
                sh '''
                  aws cloudfront create-invalidation \
                  --distribution-id E2WGKZHL0DRIXZ \
                  --paths "/*"
                '''
            }
        }
    }
}
