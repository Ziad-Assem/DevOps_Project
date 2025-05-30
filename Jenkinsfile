// pipeline {
//     agent any
//     stages {
//         stage('Access AWS Credentials') {
//             steps {
//                 withCredentials([usernamePassword(credentialsId: 'aws-access-key-id', usernameVariable: 'AWS_ACCESS_KEY_ID_FROM_JENKINS', passwordVariable: 'AWS_SECRET_ACCESS_KEY_FROM_JENKINS')]) {
//                     // The 'username' part of the credential (your AWS Access Key ID) is in AWS_ACCESS_KEY_ID_FROM_JENKINS
//                     // The 'password' part of the credential (your AWS Secret Access Key) is in AWS_SECRET_ACCESS_KEY_FROM_JENKINS

//                     // Example: Using the credentials (secure way, values will be masked in logs if echoed directly)
//                     echo "Attempting to use AWS Access Key ID: ${env.AWS_ACCESS_KEY_ID_FROM_JENKINS}" // Masked by Jenkins
//                     echo "Attempting to use AWS Secret Access Key: ${env.AWS_SECRET_ACCESS_KEY_FROM_JENKINS}" // Masked by Jenkins

//                     // For instance, you might use them with an AWS CLI command (if AWS CLI is configured on the agent)
//                     // sh 'aws s3 ls --profile temp-profile' // Assuming you configure a profile, or use env vars directly

//                     // ---------------------------------------------------------------------------------
//                     // WARNING: The following demonstrates printing the raw credentials.
//                     // THIS IS A HUGE SECURITY RISK AND SHOULD NOT BE DONE IN PRODUCTION PIPELINES.
//                     // Credentials will be exposed in your build logs.
//                     // ---------------------------------------------------------------------------------
//                     script {
//                         def accessKey = env.AWS_ACCESS_KEY_ID_FROM_JENKINS
//                         def secretKey = env.AWS_SECRET_ACCESS_KEY_FROM_JENKINS

//                         println "*****************************************************************"
//                         println "* SECURITY WARNING                        *"
//                         println "* You are about to print sensitive credentials to the console.  *"
//                         println "* This is highly discouraged and poses a significant security   *"
//                         println "* risk. Do NOT use this in production or shared environments.   *"
//                         println "*****************************************************************"
//                         println "Plain AWS Access Key ID (SECURITY RISK): ${accessKey}"
//                         println "Plain AWS Secret Access Key (SECURITY RISK): ${secretKey}"
//                         println "*****************************************************************"
//                     }
//                 }
//             }
//         }
//     }
// }

pipeline {
    agent any
    
    environment {
        AWS_DEFAULT_REGION = 'us-east-1'
        APP_NAME = 'my-app'
        TERRAFORM_VERSION = '1.5.0'
        // Docker registry will be set dynamically after getting account ID
        DOCKER_REGISTRY = ''
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
                script {
                    env.GIT_COMMIT_SHORT = sh(
                        script: "git rev-parse --short HEAD",
                        returnStdout: true
                    ).trim()
                    env.BUILD_TIMESTAMP = sh(
                        script: "date +%Y%m%d%H%M%S",
                        returnStdout: true
                    ).trim()
                    env.IMAGE_TAG = "${env.BUILD_TIMESTAMP}-${env.GIT_COMMIT_SHORT}"
                }
                echo "Building commit: ${env.GIT_COMMIT_SHORT}"
                echo "Build timestamp: ${env.BUILD_TIMESTAMP}"
            }
        }
        
        stage('Setup AWS Credentials') {
            steps {
                script {
                    // Using your existing credential ID
                    withCredentials([usernamePassword(credentialsId: 'aws-access-key-id', 
                                                    usernameVariable: 'AWS_ACCESS_KEY_ID', 
                                                    passwordVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                        // Get AWS Account ID
                        env.AWS_ACCOUNT_ID = sh(
                            script: """
                                aws sts get-caller-identity --query Account --output text
                            """,
                            returnStdout: true
                        ).trim()
                        
                        env.DOCKER_REGISTRY = "${env.AWS_ACCOUNT_ID}.dkr.ecr.${env.AWS_DEFAULT_REGION}.amazonaws.com"
                        echo "AWS Account ID: ${env.AWS_ACCOUNT_ID}"
                        echo "Docker Registry: ${env.DOCKER_REGISTRY}"
                    }
                }
            }
        }
        
        stage('Validate Terraform') {
            steps {
                script {
                    // Install Terraform if not available
                    sh '''
                        if ! terraform version; then
                            echo "Installing Terraform..."
                            wget -O terraform.zip https://releases.hashicorp.com/terraform/1.5.0/terraform_1.5.0_linux_amd64.zip
                            unzip terraform.zip
                            sudo mv terraform /usr/local/bin/
                            rm terraform.zip
                        fi
                        
                        echo "Validating Terraform configuration..."
                        terraform fmt -check=true -diff=true || true
                        terraform init -backend=false
                        terraform validate
                    '''
                }
            }
        }
        
        stage('Security Checks') {
            parallel {
                stage('Dockerfile Security Scan') {
                    steps {
                        script {
                            sh '''
                                echo "Checking if Dockerfile exists..."
                                if [ -f "Dockerfile" ]; then
                                    echo "Dockerfile found, performing basic security checks..."
                                    
                                    # Check for common security issues
                                    echo "=== Dockerfile Security Check ==="
                                    
                                    if grep -i "FROM.*:latest" Dockerfile; then
                                        echo "WARNING: Using 'latest' tag detected"
                                    fi
                                    
                                    if grep -i "RUN.*sudo" Dockerfile; then
                                        echo "WARNING: sudo usage detected"
                                    fi
                                    
                                    if grep -i "USER root" Dockerfile; then
                                        echo "WARNING: Running as root user detected"
                                    fi
                                    
                                    echo "Basic Dockerfile security check completed"
                                else
                                    echo "No Dockerfile found, skipping security scan"
                                fi
                            '''
                        }
                    }
                }
                
                stage('Terraform Security Scan') {
                    steps {
                        script {
                            sh '''
                                echo "=== Terraform Security Check ==="
                                
                                # Check for common security issues in Terraform
                                if grep -r "0.0.0.0/0" *.tf; then
                                    echo "INFO: Open security group rules detected (review if intentional)"
                                fi
                                
                                if grep -r "force_delete.*true" *.tf; then
                                    echo "INFO: Force delete enabled on resources"
                                fi
                                
                                echo "Basic Terraform security check completed"
                            '''
                        }
                    }
                }
            }
        }
        
        stage('Build Application') {
            when {
                anyOf {
                    branch 'main'
                    branch 'master'
                    branch 'develop'
                }
            }
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'aws-access-key-id', 
                                                    usernameVariable: 'AWS_ACCESS_KEY_ID', 
                                                    passwordVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                        sh '''
                            echo "=== Building Application ==="
                            
                            # Dockerfile already exists in the repo, verify it
                            if [ -f "Dockerfile" ]; then
                                echo "✅ Dockerfile found and ready to use"
                                ls -la Dockerfile
                            else
                                echo "❌ Dockerfile not found in repository"
                                exit 1
                            fi
                            
                            # Login to ECR (will be created by Terraform if it doesn't exist)
                            echo "Logging into AWS ECR..."
                            aws ecr get-login-password --region $AWS_DEFAULT_REGION | docker login --username AWS --password-stdin $DOCKER_REGISTRY || echo "ECR login failed, will be created by Terraform"
                            
                            echo "Build stage completed successfully"
                        '''
                    }
                }
            }
        }
        
        stage('Deploy Infrastructure') {
            when {
                anyOf {
                    branch 'main'
                    branch 'master'
                    branch 'develop'
                }
            }
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'aws-access-key-id', 
                                                    usernameVariable: 'AWS_ACCESS_KEY_ID', 
                                                    passwordVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                        sh '''
                            echo "=== Deploying Infrastructure with Terraform ==="
                            
                            # Initialize Terraform
                            terraform init
                            
                            # Plan the deployment
                            terraform plan -var="app_name=$APP_NAME" -var="aws_region=$AWS_DEFAULT_REGION" -out=tfplan
                            
                            # Apply the plan
                            terraform apply -auto-approve tfplan
                            
                            # Capture outputs
                            terraform output -json > terraform_outputs.json
                            
                            echo "=== Terraform Outputs ==="
                            cat terraform_outputs.json
                            
                            # Extract important values
                            ECR_URL=$(terraform output -raw ecr_repository_url)
                            ECS_CLUSTER=$(terraform output -raw ecs_cluster_name)
                            ECS_SERVICE=$(terraform output -raw ecs_service_name)
                            IMAGE_VERSION=$(terraform output -raw docker_image_version)
                            
                            echo "ECR Repository URL: $ECR_URL"
                            echo "ECS Cluster: $ECS_CLUSTER"
                            echo "ECS Service: $ECS_SERVICE"
                            echo "Image Version: $IMAGE_VERSION"
                            
                            # Save these for later stages
                            echo "$ECR_URL" > ecr_url.txt
                            echo "$ECS_CLUSTER" > ecs_cluster.txt
                            echo "$ECS_SERVICE" > ecs_service.txt
                            echo "$IMAGE_VERSION" > image_version.txt
                        '''
                    }
                }
            }
        }
        
        stage('Verify Deployment') {
            when {
                anyOf {
                    branch 'main'
                    branch 'master'
                    branch 'develop'
                }
            }
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'aws-access-key-id', 
                                                    usernameVariable: 'AWS_ACCESS_KEY_ID', 
                                                    passwordVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                        sh '''
                            echo "=== Verifying Deployment ==="
                            
                            # Read saved values
                            ECS_CLUSTER=$(cat ecs_cluster.txt)
                            ECS_SERVICE=$(cat ecs_service.txt)
                            
                            # Check ECS service status
                            echo "Checking ECS service status..."
                            aws ecs describe-services --cluster $ECS_CLUSTER --services $ECS_SERVICE --region $AWS_DEFAULT_REGION
                            
                            # Wait for service to stabilize
                            echo "Waiting for service to stabilize..."
                            aws ecs wait services-stable --cluster $ECS_CLUSTER --services $ECS_SERVICE --region $AWS_DEFAULT_REGION --timeout 600
                            
                            # Get task status
                            echo "Getting running tasks..."
                            aws ecs list-tasks --cluster $ECS_CLUSTER --service-name $ECS_SERVICE --region $AWS_DEFAULT_REGION
                            
                            echo "Deployment verification completed"
                        '''
                    }
                }
            }
        }
        
        stage('Health Check') {
            when {
                anyOf {
                    branch 'main'
                    branch 'master'
                    branch 'develop'
                }
            }
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'aws-access-key-id', 
                                                    usernameVariable: 'AWS_ACCESS_KEY_ID', 
                                                    passwordVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                        sh '''
                            echo "=== Performing Health Check ==="
                            
                            ECS_CLUSTER=$(cat ecs_cluster.txt)
                            ECS_SERVICE=$(cat ecs_service.txt)
                            
                            # Get task ARNs
                            TASK_ARNS=$(aws ecs list-tasks --cluster $ECS_CLUSTER --service-name $ECS_SERVICE --region $AWS_DEFAULT_REGION --query 'taskArns[0]' --output text)
                            
                            if [ "$TASK_ARNS" != "None" ] && [ "$TASK_ARNS" != "" ]; then
                                echo "Task ARN: $TASK_ARNS"
                                
                                # Get task details
                                aws ecs describe-tasks --cluster $ECS_CLUSTER --tasks $TASK_ARNS --region $AWS_DEFAULT_REGION
                                
                                echo "Health check completed - Service is running"
                            else
                                echo "No running tasks found"
                                exit 1
                            fi
                        '''
                    }
                }
            }
        }
    }
    
    post {
        always {
            script {
                // Clean up temporary files
                sh '''
                    rm -f tfplan terraform_outputs.json ecr_url.txt ecs_cluster.txt ecs_service.txt image_version.txt
                '''
            }
            
            // Archive Terraform state and logs
            archiveArtifacts artifacts: 'terraform.tfstate*', allowEmptyArchive: true
            
            // Clean workspace on successful builds
            cleanWs(cleanWhenAborted: false, cleanWhenFailure: false, cleanWhenSuccess: true)
        }
        
        success {
            echo "✅ Pipeline completed successfully!"
            
            // Send success notification (configure as needed)
            script {
                def commitMsg = sh(script: "git log -1 --pretty=%B", returnStdout: true).trim()
                echo "Deployed commit: ${env.GIT_COMMIT_SHORT}"
                echo "Commit message: ${commitMsg}"
            }
        }
        
        failure {
            echo "❌ Pipeline failed!"
            
            // Clean up on failure - destroy infrastructure if needed
            script {
                withCredentials([usernamePassword(credentialsId: 'aws-access-key-id', 
                                                usernameVariable: 'AWS_ACCESS_KEY_ID', 
                                                passwordVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                    sh '''
                        echo "Pipeline failed. Checking if cleanup is needed..."
                        # Uncomment the line below if you want to destroy infrastructure on failure
                        # terraform destroy -auto-approve -var="app_name=$APP_NAME" -var="aws_region=$AWS_DEFAULT_REGION"
                    '''
                }
            }
        }
        
        unstable {
            echo "⚠️ Pipeline completed with warnings"
        }
    }
}