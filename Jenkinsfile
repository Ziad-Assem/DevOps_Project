stage('Setup AWS Credentials') {
    steps {
        script {
            withCredentials([usernamePassword(credentialsId: 'aws-access-key-id', 
                                            usernameVariable: 'AWS_ACCESS_KEY_ID', 
                                            passwordVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                // Install AWS CLI if missing
                sh '''
                    if ! command -v aws &> /dev/null; then
                        echo "Installing AWS CLI..."
                        curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
                        unzip awscliv2.zip
                        sudo ./aws/install
                        rm -rf awscliv2.zip aws
                    fi
                '''
                
                // Get AWS Account ID
                env.AWS_ACCOUNT_ID = sh(
                    script: "aws sts get-caller-identity --query Account --output text",
                    returnStdout: true
                ).trim()
                
                env.DOCKER_REGISTRY = "${env.AWS_ACCOUNT_ID}.dkr.ecr.${env.AWS_DEFAULT_REGION}.amazonaws.com"
                echo "AWS Account ID: ${env.AWS_ACCOUNT_ID}"
                echo "Docker Registry: ${env.DOCKER_REGISTRY}"
            }
        }
    }
}