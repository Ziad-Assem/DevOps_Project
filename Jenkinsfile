// pipeline {
//     agent any
    
//     environment {
//         AWS_DEFAULT_REGION = 'us-east-1'
//         DOCKER_BUILDKIT = '1'
//         // Define your app name - change this to match your preference
//         APP_NAME = 'my-app'
//     }
    
//     parameters {
//         choice(
//             name: 'ACTION',
//             choices: ['apply', 'destroy'],
//             description: 'Choose whether to deploy (apply) or destroy the infrastructure'
//         )
//         string(
//             name: 'APP_NAME_OVERRIDE',
//             defaultValue: '',
//             description: 'Optional: Override the default app name'
//         )
//     }
    
//     stages {
//         stage('Checkout') {
//             steps {
//                 echo "🔄 Checking out code from GitHub..."
//                 checkout scm
                
//                 script {
//                     // Show what we've checked out
//                     sh '''
//                         echo "Repository contents:"
//                         ls -la
//                         echo "Current branch: $(git branch --show-current 2>/dev/null || echo 'detached HEAD')"
//                         echo "Latest commit: $(git log -1 --oneline 2>/dev/null || echo 'N/A')"
//                     '''
//                 }
//             }
//         }
        
//         stage('Validate Environment') {
//             steps {
//                 withCredentials([
//                     usernamePassword(
//                         credentialsId: 'aws-access-key-id',
//                         usernameVariable: 'AWS_ACCESS_KEY_ID',
//                         passwordVariable: 'AWS_SECRET_ACCESS_KEY'
//                     )
//                 ]) {
//                     script {
//                         sh '''
//                             echo "🔍 Validating environment..."
                            
//                             # Check AWS CLI
//                             echo "Testing AWS CLI access..."
//                             aws sts get-caller-identity --output text
//                             echo "✅ AWS CLI is working!"
                            
//                             # Check Terraform
//                             echo "Checking Terraform version..."
//                             terraform --version
//                             echo "✅ Terraform is available!"
                            
//                             # Check Docker
//                             echo "Checking Docker version..."
//                             docker --version
//                             echo "✅ Docker is available!"
                            
//                             # Verify required files exist
//                             echo "Verifying required files..."
//                             if [ ! -f "main.tf" ]; then
//                                 echo "❌ main.tf not found!"
//                                 exit 1
//                             fi
//                             if [ ! -f "Dockerfile" ]; then
//                                 echo "❌ Dockerfile not found!"
//                                 exit 1
//                             fi
//                             echo "✅ Required files found!"
//                         '''
//                     }
//                 }
//             }
//         }
        
//         stage('Terraform Init') {
//             steps {
//                 withCredentials([
//                     usernamePassword(
//                         credentialsId: 'aws-access-key-id',
//                         usernameVariable: 'AWS_ACCESS_KEY_ID',
//                         passwordVariable: 'AWS_SECRET_ACCESS_KEY'
//                     )
//                 ]) {
//                     script {
//                         sh '''
//                             echo "🚀 Initializing Terraform..."
//                             terraform init
//                             echo "✅ Terraform initialized successfully!"
//                         '''
//                     }
//                 }
//             }
//         }
        
//         stage('Terraform Plan') {
//             steps {
//                 withCredentials([
//                     usernamePassword(
//                         credentialsId: 'aws-access-key-id',
//                         usernameVariable: 'AWS_ACCESS_KEY_ID',
//                         passwordVariable: 'AWS_SECRET_ACCESS_KEY'
//                     )
//                 ]) {
//                     script {
//                         def appName = params.APP_NAME_OVERRIDE ?: env.APP_NAME
                        
//                         if (params.ACTION == 'apply') {
//                             sh """
//                                 echo "📋 Creating Terraform plan for deployment..."
//                                 terraform plan \\
//                                     -var="app_name=${appName}" \\
//                                     -var="aws_region=${AWS_DEFAULT_REGION}" \\
//                                     -var="docker_image_path=." \\
//                                     -out=tfplan
//                                 echo "✅ Terraform plan created successfully!"
//                             """
//                         } else {
//                             sh """
//                                 echo "📋 Creating Terraform plan for destruction..."
//                                 terraform plan \\
//                                     -var="app_name=${appName}" \\
//                                     -var="aws_region=${AWS_DEFAULT_REGION}" \\
//                                     -var="docker_image_path=." \\
//                                     -destroy \\
//                                     -out=tfplan
//                                 echo "✅ Terraform destroy plan created successfully!"
//                             """
//                         }
//                     }
//                 }
//             }
//         }
        
//         stage('Manual Approval') {
//             when {
//                 expression { params.ACTION == 'destroy' }
//             }
//             steps {
//                 script {
//                     def userInput = input(
//                         id: 'userInput',
//                         message: '⚠️  You are about to DESTROY the infrastructure. Are you sure?',
//                         parameters: [
//                             choice(
//                                 choices: ['No', 'Yes'],
//                                 description: 'Select Yes to proceed with destruction',
//                                 name: 'CONFIRM_DESTROY'
//                             )
//                         ]
//                     )
                    
//                     if (userInput != 'Yes') {
//                         error('❌ Destruction cancelled by user')
//                     }
//                 }
//             }
//         }
        
//         stage('Terraform Apply/Destroy') {
//             steps {
//                 withCredentials([
//                     usernamePassword(
//                         credentialsId: 'aws-access-key-id',
//                         usernameVariable: 'AWS_ACCESS_KEY_ID',
//                         passwordVariable: 'AWS_SECRET_ACCESS_KEY'
//                     )
//                 ]) {
//                     script {
//                         if (params.ACTION == 'apply') {
//                             sh '''
//                                 echo "🚀 Applying Terraform configuration..."
//                                 echo "This will:"
//                                 echo "  - Create ECR repository"
//                                 echo "  - Build and push Docker image"
//                                 echo "  - Create ECS cluster and service"
//                                 echo "  - Deploy your application"
                                
//                                 terraform apply tfplan
//                                 echo "✅ Terraform apply completed successfully!"
//                             '''
//                         } else {
//                             sh '''
//                                 echo "💥 Destroying Terraform infrastructure..."
//                                 terraform apply tfplan
//                                 echo "✅ Infrastructure destroyed successfully!!"
//                             '''
//                         }
//                     }
//                 }
//             }
//         }
        
//         stage('Get Outputs') {
//             when {
//                 expression { params.ACTION == 'apply' }
//             }
//             steps {
//                 withCredentials([
//                     usernamePassword(
//                         credentialsId: 'aws-access-key-id',
//                         usernameVariable: 'AWS_ACCESS_KEY_ID',
//                         passwordVariable: 'AWS_SECRET_ACCESS_KEY'
//                     )
//                 ]) {
//                     script {
//                         sh '''
//                             echo "📊 Getting Terraform outputs..."
//                             terraform output
                            
//                             echo ""
//                             echo "🎉 Deployment Summary:"
//                             echo "ECR Repository: $(terraform output -raw ecr_repository_url)"
//                             echo "ECS Cluster: $(terraform output -raw ecs_cluster_name)"
//                             echo "ECS Service: $(terraform output -raw ecs_service_name)"
//                             echo "Docker Image Version: $(terraform output -raw docker_image_version)"
//                             echo ""
//                             echo "🔗 Your application should be accessible via the ECS service endpoint"
//                             echo "📝 Check the AWS Console for the service's public IP address"
//                         '''
//                     }
//                 }
//             }
//         }
//     }
    
//     post {
//         always {
//             echo "🧹 Cleaning up workspace..."
//             // Clean up sensitive files but keep terraform state
//             sh '''
//                 rm -f tfplan
//                 echo "✅ Cleanup completed"
//             '''
//         }
//         success {
//             script {
//                 if (params.ACTION == 'apply') {
//                     echo "🎉 SUCCESS: Infrastructure deployed successfully!"
//                     echo "Your application is now running on AWS ECS Fargate"
//                 } else {
//                     echo "🎉 SUCCESS: Infrastructure destroyed successfully!"
//                 }
//             }
//         }
//         failure {
//             echo "❌ FAILURE: Pipeline failed. Check the logs above for details."
//         }
//         unstable {
//             echo "⚠️  UNSTABLE: Pipeline completed with warnings."
//         }
//     }
// }

pipeline {
    agent any
    
    environment {
        AWS_DEFAULT_REGION = 'us-east-1'
        DOCKER_BUILDKIT = '1'
        // Define your app name - change this to match your preference
        APP_NAME = 'my-app'
    }
    
    parameters {
        choice(
            name: 'ACTION',
            choices: ['apply', 'destroy'],
            description: 'Choose whether to deploy (apply) or destroy the infrastructure'
        )
        string(
            name: 'APP_NAME_OVERRIDE',
            defaultValue: '',
            description: 'Optional: Override the default app name'
        )
    }
    
    stages {
        stage('Checkout') {
            steps {
                echo "🔄 Checking out code from GitHub..."
                checkout scm
                
                script {
                    // Show what we've checked out
                    sh '''
                        echo "Repository contents:"
                        ls -la
                        echo "Current branch: $(git branch --show-current 2>/dev/null || echo 'detached HEAD')"
                        echo "Latest commit: $(git log -1 --oneline 2>/dev/null || echo 'N/A')"
                    '''
                }
            }
        }
        
        stage('Validate Environment') {
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
                            echo "🔍 Validating environment..."
                            
                            # Check AWS CLI
                            echo "Testing AWS CLI access..."
                            aws sts get-caller-identity --output text
                            echo "✅ AWS CLI is working!"
                            
                            # Check Terraform
                            echo "Checking Terraform version..."
                            terraform --version
                            echo "✅ Terraform is available!"
                            
                            # Check Docker
                            echo "Checking Docker version..."
                            docker --version
                            echo "✅ Docker is available!"
                            
                            # Verify required files exist
                            echo "Verifying required files..."
                            if [ ! -f "main.tf" ]; then
                                echo "❌ main.tf not found!"
                                exit 1
                            fi
                            if [ ! -f "Dockerfile" ]; then
                                echo "❌ Dockerfile not found!"
                                exit 1
                            fi
                            echo "✅ Required files found!"
                        '''
                    }
                }
            }
        }
        
        stage('Terraform Init') {
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
                            echo "🚀 Initializing Terraform..."
                            terraform init
                            echo "✅ Terraform initialized successfully!"
                        '''
                    }
                }
            }
        }
        
        stage('Terraform Plan') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'aws-access-key-id',
                        usernameVariable: 'AWS_ACCESS_KEY_ID',
                        passwordVariable: 'AWS_SECRET_ACCESS_KEY'
                    )
                ]) {
                    script {
                        def appName = params.APP_NAME_OVERRIDE ?: env.APP_NAME
                        
                        if (params.ACTION == 'apply') {
                            sh """
                                echo "📋 Creating Terraform plan for deployment..."
                                terraform plan \\
                                    -var="app_name=${appName}" \\
                                    -var="aws_region=${AWS_DEFAULT_REGION}" \\
                                    -var="docker_image_path=." \\
                                    -out=tfplan
                                echo "✅ Terraform plan created successfully!"
                            """
                        } else {
                            sh """
                                echo "📋 Creating Terraform plan for destruction..."
                                terraform plan \\
                                    -var="app_name=${appName}" \\
                                    -var="aws_region=${AWS_DEFAULT_REGION}" \\
                                    -var="docker_image_path=." \\
                                    -destroy \\
                                    -out=tfplan
                                echo "✅ Terraform destroy plan created successfully!"
                            """
                        }
                    }
                }
            }
        }
        
        stage('Manual Approval') {
            when {
                expression { params.ACTION == 'destroy' }
            }
            steps {
                script {
                    def userInput = input(
                        id: 'userInput',
                        message: '⚠️  You are about to DESTROY the infrastructure. Are you sure?',
                        parameters: [
                            choice(
                                choices: ['No', 'Yes'],
                                description: 'Select Yes to proceed with destruction',
                                name: 'CONFIRM_DESTROY'
                            )
                        ]
                    )
                    
                    if (userInput != 'Yes') {
                        error('❌ Destruction cancelled by user')
                    }
                }
            }
        }
        
        stage('Terraform Apply/Destroy') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'aws-access-key-id',
                        usernameVariable: 'AWS_ACCESS_KEY_ID',
                        passwordVariable: 'AWS_SECRET_ACCESS_KEY'
                    )
                ]) {
                    script {
                        if (params.ACTION == 'apply') {
                            sh '''
                                echo "🚀 Applying Terraform configuration..."
                                echo "This will:"
                                echo "  - Create ECR repository"
                                echo "  - Build and push Docker image"
                                echo "  - Create ECS cluster and service"
                                echo "  - Deploy your application"
                                
                                # Apply with auto-approve and detailed logging
                                terraform apply -auto-approve tfplan
                                echo "✅ Terraform apply completed successfully!"
                            '''
                        } else {
                            sh '''
                                echo "💥 Destroying Terraform infrastructure..."
                                terraform apply -auto-approve tfplan
                                echo "✅ Infrastructure destroyed successfully!"
                            '''
                        }
                    }
                }
            }
        }
        
        stage('Verify Deployment') {
            when {
                expression { params.ACTION == 'apply' }
            }
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
                            echo "🔍 Verifying deployment..."
                            
                            # Get the ECS service name and cluster
                            SERVICE_NAME=$(terraform output -raw ecs_service_name)
                            CLUSTER_NAME=$(terraform output -raw ecs_cluster_name)
                            IMAGE_URI=$(terraform output -raw docker_image_uri)
                            
                            echo "Service: $SERVICE_NAME"
                            echo "Cluster: $CLUSTER_NAME"
                            echo "Image: $IMAGE_URI"
                            
                            # Check if image exists in ECR
                            echo "Verifying image in ECR..."
                            aws ecr describe-images --repository-name $(terraform output -raw app_name) --region ${AWS_DEFAULT_REGION} | grep -q "$(terraform output -raw docker_image_version)"
                            
                            if [ $? -eq 0 ]; then
                                echo "✅ Image verified in ECR"
                            else
                                echo "❌ Image not found in ECR"
                                exit 1
                            fi
                            
                            # Wait for service to be stable
                            echo "Waiting for ECS service to stabilize..."
                            aws ecs wait services-stable --cluster $CLUSTER_NAME --services $SERVICE_NAME --region ${AWS_DEFAULT_REGION}
                            
                            # Check service status
                            echo "Checking service status..."
                            aws ecs describe-services --cluster $CLUSTER_NAME --services $SERVICE_NAME --region ${AWS_DEFAULT_REGION} --query 'services[0].{Status:status,Running:runningCount,Desired:desiredCount,TaskDefinition:taskDefinition}'
                            
                            echo "✅ Deployment verification completed!"
                        '''
                    }
                }
            }
        }
        
        stage('Get Outputs') {
            when {
                expression { params.ACTION == 'apply' }
            }
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
                            echo "📊 Getting Terraform outputs..."
                            terraform output
                            
                            echo ""
                            echo "🎉 Deployment Summary:"
                            echo "ECR Repository: $(terraform output -raw ecr_repository_url)"
                            echo "ECS Cluster: $(terraform output -raw ecs_cluster_name)"
                            echo "ECS Service: $(terraform output -raw ecs_service_name)"
                            echo "Docker Image Version: $(terraform output -raw docker_image_version)"
                            echo "Docker Image URI: $(terraform output -raw docker_image_uri)"
                            
                            # Get the public IP of the running task
                            echo ""
                            echo "🔗 Getting service endpoint..."
                            CLUSTER_NAME=$(terraform output -raw ecs_cluster_name)
                            SERVICE_NAME=$(terraform output -raw ecs_service_name)
                            
                            # Get task ARN
                            TASK_ARN=$(aws ecs list-tasks --cluster $CLUSTER_NAME --service-name $SERVICE_NAME --query 'taskArns[0]' --output text --region ${AWS_DEFAULT_REGION})
                            
                            if [ "$TASK_ARN" != "None" ] && [ "$TASK_ARN" != "" ]; then
                                # Get ENI ID
                                ENI_ID=$(aws ecs describe-tasks --cluster $CLUSTER_NAME --tasks $TASK_ARN --query 'tasks[0].attachments[0].details[?name==`networkInterfaceId`].value' --output text --region ${AWS_DEFAULT_REGION})
                                
                                if [ "$ENI_ID" != "" ]; then
                                    # Get public IP
                                    PUBLIC_IP=$(aws ec2 describe-network-interfaces --network-interface-ids $ENI_ID --query 'NetworkInterfaces[0].Association.PublicIp' --output text --region ${AWS_DEFAULT_REGION})
                                    
                                    if [ "$PUBLIC_IP" != "None" ] && [ "$PUBLIC_IP" != "" ]; then
                                        echo "🌐 Your application is accessible at: http://$PUBLIC_IP"
                                    else
                                        echo "⚠️  Public IP not yet assigned or not available"
                                    fi
                                else
                                    echo "⚠️  ENI ID not found"
                                fi
                            else
                                echo "⚠️  No running tasks found yet"
                            fi
                            
                            echo ""
                            echo "📝 You can also check the AWS Console ECS service for the current status"
                        '''
                    }
                }
            }
        }
    }
    
    post {
        always {
            echo "🧹 Cleaning up workspace..."
            // Clean up sensitive files but keep terraform state
            sh '''
                rm -f tfplan
                echo "✅ Cleanup completed"
            '''
        }
        success {
            script {
                if (params.ACTION == 'apply') {
                    echo "🎉 SUCCESS: Infrastructure deployed successfully!"
                    echo "Your application is now running on AWS ECS Fargate"
                } else {
                    echo "🎉 SUCCESS: Infrastructure destroyed successfully!"
                }
            }
        }
        failure {
            echo "❌ FAILURE: Pipeline failed. Check the logs above for details."
        }
        unstable {
            echo "⚠️  UNSTABLE: Pipeline completed with warnings."
        }
    }
}