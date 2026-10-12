pipeline {
    agent any

    parameters {
        choice(
            name: 'ENVIRONMENT',
            choices: ['dev', 'stg', 'prod'],
            description: 'Select the Terraform environment'
        )

        booleanParam(
            name: 'APPROVE_APPLY',
            defaultValue: false,
            description: 'Approve Terraform apply'
        )
    }

    environment {
        TF_IN_AUTOMATION = 'true'
        TF_INPUT = 'false'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Terraform Init') {
            steps {
                sh 'terraform init'
            }
        }

        stage('Workspace Select') {
            steps {
                script {
                    sh """
                        terraform workspace select ${params.ENVIRONMENT} || \
                        terraform workspace new ${params.ENVIRONMENT}
                    """
                }
            }
        }

        stage('Plan') {
            steps {
                script {
                    def varFile = "${params.ENVIRONMENT}.tfvars"

                    sh """
                        terraform plan \
                          -var-file=${varFile} \
                          -out=tfplan
                    """
                }
            }
        }

        stage('Approve') {
            steps {
                script {
                    if (params.ENVIRONMENT == 'prod') {
                        input(
                            message: 'Approve Terraform deployment to PROD?',
                            ok: 'Approve'
                        )
                    } else if (params.APPROVE_APPLY) {
                        echo 'Terraform apply approved.'
                    } else {
                        input(
                            message: 'Review the Terraform plan before applying.',
                            ok: 'Approve'
                        )
                    }
                }
            }
        }

        stage('Apply') {
            steps {
                sh 'terraform apply -auto-approve tfplan'
            }
        }
    }

    post {
        success {
            echo 'Terraform pipeline completed successfully.'
        }

        failure {
            mail(
                to: 'admin@example.com',
                subject: "Terraform Pipeline Failed: ${env.JOB_NAME}",
                body: """
                    Job: ${env.JOB_NAME}
                    Build: ${env.BUILD_NUMBER}
                    URL: ${env.BUILD_URL}
                    Status: FAILED
                """
            )
        }

        always {
            echo 'Pipeline execution finished.'
        }
    }
}