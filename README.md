Azure Linux Web App Deployment with Terraform & Azure DevOps PipelinesAn end-to-end Infrastructure as Code (IaC) project automating the deployment of a managed Linux Web App on Microsoft Azure using Terraform and multi-stage Azure DevOps CI/CD Pipelines.

📋 Table of Contents
   1. Architecture Overview
   2. Project Structure
   3. Prerequisites
   4. Setup & Deployment Steps
   5. CI/CD Pipeline Workflow
   6. Troubleshooting & Lessons Learned
   
   
🏛️ Architecture Overview
   
   [ Developer / Git Push ]
          │
          ▼
   [ GitHub Repo ] ──(Triggers)──► [ Azure DevOps Pipeline ]
                                          │
                               (Authenticates via SPN)
                                          │
                                          ▼
                               [ Azure Cloud Platform ]
                              ┌───────────┴───────────┐
                              ▼                       ▼
                     [ Azure Storage ]       [ Azure Web App ]
                     (Remote State File)    (Host Application)

                     
This project provisions and manages the following components:
  1.Infrastructure as Code (IaC): Terraform scripts written in HCL to declare cloud resources dynamically.
  2.Remote State Management: Azure Blob Storage account (tfstatedemostg5) handling state locking and drift prevention.
  3.CI/CD Automation: A two-stage Azure DevOps YAML pipeline running on ubuntu-latest agents.
  4.Security & Access Control: Azure Entra ID Service Principal with Federated Credentials enforcing Least-Privilege RBAC.
  5.Provisioned Resources:
     * Azure Resource Group (rg-devops-project-20)
     * Azure App Service Plan (asp-project20-free)
     * Azure Linux Web App (app-nithya-devops-20)
     
📁 Project StructurePlaintext.

├── Azure DevOps pipeline + Terraform Deployment/
│   ├── main.tf              # Defines Azure Provider, Resource Group, App Service Plan, and Linux Web App
│   ├── variables.tf         # Variable declarations (Region, RG Name, SKU configurations)
│   ├── terraform.tfvars     # Environment-specific parameter values
│   └── azure-pipelines.yml  # Multi-stage CI/CD pipeline definition
└── README.md                # Project documentation


⚡ Prerequisites
Before deploying this pipeline, ensure you have:
1.An active Microsoft Azure Subscription.
2.An Azure DevOps Organization linked to your repository.
3.An Azure Entra ID Service Principal (tfdemo-spn) configured with a Service Connection (azure-spn-conn) in Azure DevOps using Federated Credentials.
4.An Azure Storage Account and Container (tfstate) created for Terraform remote state backend storage.

🚀 Setup & Deployment Steps
1. Clone the RepositoryBashgit clone https://github.com/YOUR_USERNAME/YOUR_REPOSITORY_NAME.git
cd "YOUR_REPOSITORY_NAME/Azure DevOps pipeline + Terraform Deployment"
2. Configure Backend & VariablesVerify your backend configurations in main.tf match your target storage account:
Terraform
terraform {
  backend "azurerm" {
    resource_group_name  = "rg-devops-project-20"
    storage_account_name = "tfstatedemostg5"
    container_name       = "tfstate"
    key                  = "project20.terraform.tfstate"
  }
}
Ensure terraform.tfvars reflects your target deployment region:
Terraform
resource_group_name = "rg-devops-project-20"
location            = "Central US"

🔄 CI/CD Pipeline Workflow
The pipeline defined in azure-pipelines.yml executes automatically upon pushing changes to main or master branches:
Stage 1: 
    TerraformPlanInstalls Terraform CLI on the Microsoft-hosted Ubuntu runner.
    Runs terraform init to initialize the remote backend.
    Executes terraform validate to check HCL syntax.
    Runs terraform plan to log proposed infrastructure updates.
Stage 2: 
    TerraformApplyTriggered upon successful completion of Stage 1.
    Runs terraform init and applies infrastructure changes directly via terraform apply -auto-approve.
    
🛠️ Troubleshooting & Lessons Learned
Challenge/Error 
LogCauseSolutionAADSTS70025: No configured federated identityService principal lacked federated trust credentials for Azure DevOps authentication.
Configured Federated Credentials on tfdemo-spn matching the DevOps Organization and Service Connection name.

Error: Saved plan is staleAttempting to pass binary tfplan artifacts across ephemeral runners resulted in serial mismatches upon initialization.Refactored pipeline to perform terraform apply -auto-approve dynamically in Stage 2.

Quota Exceeded (F1 VMs: 0)Regional capacity limits on free-tier App Services in East US.Updated deployment region to Central US in variables.tf and terraform.tfvars.

always_on cannot be set to true when using Free, F1 SKUAzure policy restriction on shared free tiers (F1/D1).Updated site_config block in main.tf to set always_on = false.

🤝 Contributing & ContactFeel free to open issues or submit pull requests for enhancements!

Author: Nithya
LinkedIn:https://www.linkedin.com/in/nithya-sri-palanivel-427440213/
