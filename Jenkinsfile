pipeline {
  agent any

  options {
    disableConcurrentBuilds()
    timestamps()
  }

  parameters {
    choice(name: 'ACTION', choices: ['apply', 'destroy'], description: 'Choose whether to create or destroy infrastructure')
  }

  environment {
    TF_IN_AUTOMATION  = 'true'
    AWS_DEFAULT_REGION = 'ca-central-1'
    AWS_ACCESS_KEY_ID = credentials('aws-access-key-id')
    AWS_SECRET_ACCESS_KEY = credentials('aws-secret-access-key')
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Terraform Version') {
      steps {
        sh 'terraform version'
      }
    }

    stage('Terraform Init') {
      steps {
        dir('terraform') {
          sh 'terraform init -input=false'
        }
      }
    }

    stage('Terraform Format Check') {
      steps {
        dir('terraform') {
          sh 'terraform fmt -check'
        }
      }
    }

    stage('Terraform Validate') {
      steps {
        dir('terraform') {
          sh 'terraform validate'
        }
      }
    }

    stage('Terraform Plan') {
      steps {
        dir('terraform') {
          sh '''
            if [ "${ACTION}" = "destroy" ]; then
              terraform plan -destroy -out=tfplan -input=false
            else
              terraform plan -out=tfplan -input=false
            fi
          '''
        }
      }
    }

    stage('Terraform Apply/Destroy') {
      steps {
        dir('terraform') {
          sh 'terraform apply -auto-approve -input=false tfplan'
        }
      }
    }

    stage('Show Outputs') {
      when {
        expression { params.ACTION == 'apply' }
      }
      steps {
        dir('terraform') {
          sh 'terraform output'
        }
      }
    }
  }

  post {
    success {
      echo "Pipeline completed successfully with ACTION=${params.ACTION}."
    }
    failure {
      echo 'Pipeline failed. Check stage logs for details.'
    }
  }
}
