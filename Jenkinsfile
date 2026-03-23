pipeline {
    agent any

    parameters {
        string(name: 'INSTANCE_ID', defaultValue: '', description: 'Enter EC2 Instance ID')
        choice(name: 'ACTION', choices: ['start', 'stop', 'reboot', 'terminate'], description: 'Action to perform')
        choice(name: 'ENV', choices: ['prod', 'non-prod'], description: 'Environment')
    }

    stages {

        stage('Validate Environment') {
            steps {
                script {
                    // Block all actions for production
                    if (params.ENV == 'prod') {
                        error("❌ No changes allowed in PROD environment")
                    }
                }
            }
        }

        stage('Get Instance State') {
            steps {
                script {
                    // Fetch instance state
                    def state = sh(
                        script: "aws ec2 describe-instances --instance-ids ${params.INSTANCE_ID} --query 'Reservations[0].Instances[0].State.Name' --output text",
                        returnStdout: true
                    ).trim()

                    echo "Instance current state: ${state}"

                    // Save state for later stages
                    env.INSTANCE_STATE = state
                }
            }
        }

        stage('Perform Action') {
            steps {
                script {

                    if (params.ACTION == 'start' && env.INSTANCE_STATE == 'stopped') {
                        sh "aws ec2 start-instances --instance-ids ${params.INSTANCE_ID}"
                        echo "✅ Instance started"
                    }

                    else if (params.ACTION == 'stop' && env.INSTANCE_STATE == 'running') {
                        sh "aws ec2 stop-instances --instance-ids ${params.INSTANCE_ID}"
                        echo "✅ Instance stopped"
                    }

                    else if (params.ACTION == 'reboot' && env.INSTANCE_STATE == 'running') {
                        sh "aws ec2 reboot-instances --instance-ids ${params.INSTANCE_ID}"
                        echo "🔄 Instance rebooted"
                    }

                    else if (params.ACTION == 'terminate') {
                        sh "aws ec2 terminate-instances --instance-ids ${params.INSTANCE_ID}"
                        echo "💀 Instance terminated"
                    }

                    else {
                        echo "⚠️ Invalid action for current state"
                    }
                }
            }
        }
    }
}