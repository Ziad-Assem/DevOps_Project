// pipeline {
//     agent any
    
//     environment {
//         AWS_DEFAULT_REGION = 'us-east-1'
//         APP_NAME = 'my-app'
//         TERRAFORM_VERSION = '1.5.0'
//         DOCKER_REGISTRY = ''
//     }
    
//     stages {
//         stage('Setup AWS Credentials') {
//             steps {
//                 script {
//                     withCredentials([usernamePassword(credentialsId: 'aws-access-key-id', 
//                                                     usernameVariable: 'AWS_ACCESS_KEY_ID', 
//                                                     passwordVariable: 'AWS_SECRET_ACCESS_KEY')]) {
//                         // Install AWS CLI if missing
//                         sh '''
//                             if ! command -v aws &> /dev/null; then
//                                 echo "Installing AWS CLI..."
//                                 curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
//                                 unzip awscliv2.zip
//                                 sudo ./aws/install
//                                 rm -rf awscliv2.zip aws
//                             fi
//                         '''
                        
//                         // Get AWS Account ID
//                         env.AWS_ACCOUNT_ID = sh(
//                             script: "aws sts get-caller-identity --query Account --output text",
//                             returnStdout: true
//                         ).trim()
                        
//                         env.DOCKER_REGISTRY = "${env.AWS_ACCOUNT_ID}.dkr.ecr.${env.AWS_DEFAULT_REGION}.amazonaws.com"
//                         echo "AWS Account ID: ${env.AWS_ACCOUNT_ID}"
//                         echo "Docker Registry: ${env.DOCKER_REGISTRY}"
//                     }
//                 }
//             }
//         }
        
//         // ADD YOUR OTHER STAGES HERE
//         // stage('Checkout') { ... }
//         // stage('Validate Terraform') { ... }
//         // etc...
//     }
    
// post {
//     always {
//         script {
//             // Clean up temporary files
//             sh 'rm -f tfplan terraform_outputs.json ecr_url.txt ecs_cluster.txt ecs_service.txt image_version.txt'
//             // Archive Terraform state and logs
//             archiveArtifacts artifacts: 'terraform.tfstate*', allowEmptyArchive: true
//         }
//     }
    
//     success {
//         echo "✅ Pipeline completed successfully!"
//         script {
//             def commitMsg = sh(script: "git log -1 --pretty=%B", returnStdout: true).trim()
//             echo "Deployed commit: ${env.GIT_COMMIT_SHORT}"
//             echo "Commit message: ${commitMsg}"
//         }
//     }
    
//     failure {
//         echo "❌ Pipeline failed!"
//         script {
//             withCredentials([usernamePassword(credentialsId: 'aws-access-key-id', 
//                                             usernameVariable: 'AWS_ACCESS_KEY_ID', 
//                                             passwordVariable: 'AWS_SECRET_ACCESS_KEY')]) {
//                 sh '''
//                     echo "Pipeline failed. Checking if cleanup is needed..."
//                     # Uncomment below if you want to destroy infrastructure on failure
//                     # terraform destroy -auto-approve -var="app_name=$APP_NAME" -var="aws_region=$AWS_DEFAULT_REGION"
//                 '''
//             }
//         }
//     }
    
//     unstable {
//         echo "⚠️ Pipeline completed with warnings"
//     }
// }
// }

pipeline {
    agent any

    environment {
        // Use withCredentials to bind AWS credentials to env vars
        AWS_DEFAULT_REGION = 'us-east-1'  // Change as needed
    }

    stages {
        stage('Test AWS CLI Access') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'aws-access-key-id',
                        usernameVariable: 'AWS_ACCESS_KEY_ID',
                        passwordVariable: 'AWS_SECRET_ACCESS_KEY'
                    )
                ]) {
                    script {
                        sh '''
                            echo "Testing AWS CLI access..."
                            aws sts get-caller-identity --output text
                            echo "AWS CLI is working! ✅"

                            echo "Listing S3 buckets (if any)..."
                            aws s3 ls || echo "No S3 buckets found (or permissions issue)."
                        '''
                    }
                }
            }
        }
        stage('Check terraform Version') {
            steps {
                script {
                    sh '''
                        echo "Checking Terraform version..."
                        if ! command -v terraform &> /dev/null; then
                            echo "Terraform is not installed. Please install Terraform."
                            exit 1
                        fi
                        terraform version
                    '''
                }
    }

    post {
        always {
            echo "AWS CLI test completed."
        }
    }
}