# kxi

This repository contains Terraform code to provision a comprehensive infrastructure environment on AWS. The infrastructure includes networking components, an EKS cluster, and a bastion host for secure access to the cluster.

## Repository Structure

- **agent/**: Docker image for deploying the infrastructure from a local machine or a server (e.g., CI/CD).
- **modules/**: Terraform modules used for organizing and modularizing infrastructure components.
- **post-provisioning/**: Scripts for post-provisioning tasks specific to the EKS cluster.
- **scripts/**: Main deployment script 'kxi.sh'.
- **terraform/**: The main Terraform codebase organized by components.
  - **eks/**: Terraform code for provisioning the EKS cluster.
  - **networking/**: Terraform code for setting up networking components (VPC, subnets, etc.).
  - **bastion/**: Terraform code for provisioning the bastion host.
- **client_config.env**: Configuration file that users fill out before running the deployment script.

## Getting Started

1. **Fill out Configuration:**
   - Open `client_config.env` and fill out the necessary configurations based on your requirements. This includes, AWS credentials, terraform backend configuration and other components configuration.

1. **Build docker file:** 
    - This will be required to have a consitent environment for deploying infrastructure
     ```bash
        docker build -t terraform-kxi:latest agent/
     ```
2. **Run the Deployment Script:**
   - Once above image is built, Execute the deployment script in the `scripts/` directory, with following command:
     ```bash
     docker run -v "$(pwd):/terraform" terraform-kxi <CONFIG_FILE> <CLOUD> <OPERATION> <COMPONENT>
     ```
     eg:
     ```bash
      docker run -v "$(pwd):/terraform" terraform-kxi bash /terraform/scripts/kxi.sh ./client_config.env aws deploy k8s
     ```

3. **Steps to setup infrastructure:**
   - Run below commands in this order to deploy infrastructure
     ```bash
     docker run -v "$(pwd):/terraform" terraform-kxi bash /terraform/scripts/kxi.sh ./client_config.env aws deploy networking
     docker run -v "$(pwd):/terraform" terraform-kxi bash /terraform/scripts/kxi.sh ./client_config.env aws deploy k8s
     docker run -v "$(pwd):/terraform" terraform-kxi bash /terraform/scripts/kxi.sh ./client_config.env aws deploy post_provisioning
     docker run -v "$(pwd):/terraform" terraform-kxi bash /terraform/scripts/kxi.sh ./client_config.env aws deploy bastion
     ```
4. **Steps to tear down infrastructure:**
   - Run below commands in this order to deploy infrastructure
     ```bash
     docker run -v "$(pwd):/terraform" terraform-kxi bash /terraform/scripts/kxi.sh ./client_config.env aws destroy bastion
     docker run -v "$(pwd):/terraform" terraform-kxi bash /terraform/scripts/kxi.sh ./client_config.env aws destroy post_provisioning
     docker run -v "$(pwd):/terraform" terraform-kxi bash /terraform/scripts/kxi.sh ./client_config.env aws destroy k8s
     docker run -v "$(pwd):/terraform" terraform-kxi bash /terraform/scripts/kxi.sh ./client_config.env aws destroy networking
     ```
5. **Steps to get kube config, in order to access k8s cluster:**
   - Run below commands in this order to deploy infrastructure
   ```
      aws eks update-kubeconfig --name <CLUSTER_NAME> --region <REGION>
   ```
   Note: By default, the user/role will have full access to k8s cluster, who has created it. Update aws-auth config map, in order to allow other iam user/roles to access eks cluster.


## Additional Information

- The `agent/` directory contains a Docker image that can be used to deploy the infrastructure from a local machine or a server.
- Terraform modules in the `modules/` directory promote modularity and reusability in the infrastructure code.
- Post-provisioning scripts in `post-provisioning/` handle tasks specific to the EKS cluster after the initial provisioning.
