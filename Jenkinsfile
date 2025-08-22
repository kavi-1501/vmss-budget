pipeline {
  agent any

  parameters {
    choice(
      name: 'ACTION',
      choices: ['apply', 'destroy_budget', 'rmstate_budget'],
      description: 'apply = create VMSS + Budget; destroy_budget = delete only budget in Azure; rmstate_budget = remove budget from state only'
    )
  }

  environment {
    ARM_CLIENT_ID       = '24d1d59e-6434-45a2-8dd4-9d62fa3aa9bb'
    ARM_CLIENT_SECRET   = 'Wap8Q~FUsFAElD-GU-kpOxUUgaGlaeln7NjBTbKE'
    ARM_TENANT_ID       = '7597b8dc-1559-4285-901d-856b7ac20009'
    ARM_SUBSCRIPTION_ID = '6babc3f8-793c-4200-9aa3-51e8d33ff572'
    TF_VAR_subscription_id = '/subscriptions/6babc3f8-793c-4200-9aa3-51e8d33ff572'
  }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Terraform Init and Validate') {
      steps {
        sh '''
          terraform -chdir=terraform init -input=false
          terraform -chdir=terraform fmt -check -diff || true
          terraform -chdir=terraform validate
        '''
      }
    }

    stage('Execute Terraform') {
      steps {
        script {
          if (params.ACTION == 'apply') {
            sh '''
              terraform -chdir=terraform plan -out=tfplan
              terraform -chdir=terraform apply -auto-approve tfplan
            '''
          } else if (params.ACTION == 'destroy_budget') {
            sh '''
              terraform -chdir=terraform destroy -auto-approve -target=azurerm_consumption_budget_subscription.vmss_budget
            '''
          } else if (params.ACTION == 'rmstate_budget') {
            sh '''
              terraform -chdir=terraform state rm azurerm_consumption_budget_subscription.vmss_budget || true
            '''
          }
        }
      }
    }
  }
}
