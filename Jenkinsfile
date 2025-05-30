pipeline {
    agent any
    stages {
        stage('Access AWS Credentials') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'aws-access-key-id', usernameVariable: 'AWS_ACCESS_KEY_ID_FROM_JENKINS', passwordVariable: 'AWS_SECRET_ACCESS_KEY_FROM_JENKINS')]) {
                    // The 'username' part of the credential (your AWS Access Key ID) is in AWS_ACCESS_KEY_ID_FROM_JENKINS
                    // The 'password' part of the credential (your AWS Secret Access Key) is in AWS_SECRET_ACCESS_KEY_FROM_JENKINS

                    // Example: Using the credentials (secure way, values will be masked in logs if echoed directly)
                    echo "Attempting to use AWS Access Key ID: ${env.AWS_ACCESS_KEY_ID_FROM_JENKINS}" // Masked by Jenkins
                    echo "Attempting to use AWS Secret Access Key: ${env.AWS_SECRET_ACCESS_KEY_FROM_JENKINS}" // Masked by Jenkins

                    // For instance, you might use them with an AWS CLI command (if AWS CLI is configured on the agent)
                    // sh 'aws s3 ls --profile temp-profile' // Assuming you configure a profile, or use env vars directly

                    // ---------------------------------------------------------------------------------
                    // WARNING: The following demonstrates printing the raw credentials.
                    // THIS IS A HUGE SECURITY RISK AND SHOULD NOT BE DONE IN PRODUCTION PIPELINES.
                    // Credentials will be exposed in your build logs.
                    // ---------------------------------------------------------------------------------
                    script {
                        def accessKey = env.AWS_ACCESS_KEY_ID_FROM_JENKINS
                        def secretKey = env.AWS_SECRET_ACCESS_KEY_FROM_JENKINS

                        println "*****************************************************************"
                        println "* SECURITY WARNING                        *"
                        println "* You are about to print sensitive credentials to the console.  *"
                        println "* This is highly discouraged and poses a significant security   *"
                        println "* risk. Do NOT use this in production or shared environments.   *"
                        println "*****************************************************************"
                        println "Plain AWS Access Key ID (SECURITY RISK): ${accessKey}"
                        println "Plain AWS Secret Access Key (SECURITY RISK): ${secretKey}"
                        println "*****************************************************************"
                    }
                }
            }
        }
    }
}