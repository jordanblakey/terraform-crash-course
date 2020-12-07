# Terraform Crash Course

Terraform is a provisioning tool. Provisioning has 2 basic problems:
Day 1: We have nothing. Initial setup.
Day 2+: Evolving existing infrastructure. Adding new services.

## Infrastructure as Code

Declaratively define your infrastructure in a set of Terraform config files. Allows referencing infrastructure item config fields between items. Examples of infrastructure:

- a Virtual Private Cloud (VPC)
- Network security rules
- set of Virtual Machines (VMs)
- Virtual Machine monitors
- Load Balancers (LBs).
- DNS records
- CDN

Terraform accepts a set of configuration files. Resource Graph: TF builds a Resource Graph of what exists (State). Terraform generates a prescriptive Execution Plan for review. Change Automation: TF executes the plan, avoiding human error.

```sh
terraform refresh # Reconciles Terraforms View with the real world. "What's actually running"
terraform plan # Addresses the gap between the current view and the infrastructure we'd like to have (Desired config).
terraform apply # Starts with the plan/desired config and execute the infrastructure changes in the real world.
terraform destroy # Executes a plan to destroy all the elements/resources associated with an application.
```

## Providers (100s of Things Terraform can manage)

- SaaS (High-Level): - Datadog/Fastly/Github
- PaaS (Mid-Level): - Heroku/Heroku/Kubernetes/Lambdas
- IaaS (Low-Level): - Cloud Providers: Azure, AWS, GCP
- On Premises: OpenStack/VMware

Collaboration: Terraform plans are kept in a VCS, which is used to run a TerraForm Enterprise instance, which is a place to keep State and ENV vars encrypted.

## Terraform Modules

A "Black Box" of infrastructure supporting an application/service. Accepts inputs but abstracts away the details of the infrastructure to the consumer (for example, newly onboarded devs). Changes to modules are stored in a registry, for exmaple registry.terraform.io. Pivate registries are possible in TF Enterprise.

Terraform uses its own language, **HashiCorp Configuration Language** or **HCL**, for .tf configuration files.
Below is an example of HCL syntax, defining a *provider* (AWS) and *resource* (EC2 instance).

```tf
provider "aws" {
  access_key = "ACCESS_KEY"
  secret_key = "SECRET_KEY"
  region     = "us-east-1"
}

resource "aws_instance" "example" {
  ami           = "ami-2757f631"
  instance_type = "t2.micro"
}
```

Terraform knows how to use the APIs of different cloud platforms through providers. Typically the process of adding a platform to your configuration involves finding the provider on terraform.io, and adding a block to define if in our configuration files.

## Install Terraform on Ubuntu 20

```sh
# Add repo GPG key
curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo apt-key add -

# Add repository for terraform releases
sudo apt-add-repository "deb [arch=amd64] https://apt.releases.hashicorp.com $(lsb_release -cs) main" sudo apt update && sudo apt install terraform terraform -help terraform -help terraform -install-autocomplete

# COMMON COMMANDS
terraform apply # Builds or changes infrastructure
terraform console # Interactive console for Terraform interpolations
terraform destroy # Destroy Terraform-managed infrastructure
terraform env # Workspace management
terraform fmt # Rewrites config files to canonical format
terraform get # Download and install modules for the configuration
terraform graph # Create a visual graph of Terraform resources
terraform import # Import existing infrastructure into Terraform
terraform init # Initialize a Terraform working directory
terraform login # Obtain and save credentials for a remote host
terraform logout # remove locally-stored credentials for a remote host
terraform output # Read an output from a state file
terraform plan # Generate and show an execution plan
terraform providers # Prints a tree of the providers used in the configuration
terraform refresh # Update local state file against real resources
terraform show # Inspect Terraform state or plan
terraform taint # Manually mark a resource for recreation
terraform untaint # Manually unmark a resource as tainted
terraform validate # Validates the Terraform files
terraform version # Prints the Terraform version
terraform workspace # Workspace management

## Advanced Commands
terraform 0.12upgrade | 0.13upgrade | debug | force-unlock | push | state
```

## Terraform Workflow

A simple workflow for deployment will follow 4 basic steps:

1. Define Scope - Identify what *resources* need to be created for a given project.
2. Author Infrastructure as Code - Create configureation files in HCL based on scoped resources
3. Initialize - Run `terraform init` in the project directory with the HCL configuration files. This will download the correct provider plug-ins for the project.
4. Plan & Apply - Run `terraform plan` to verify the resource creation process and then `terraform apply` to create real resources as well as a state file that compares future changes in your config files to what actually exists in your deployment envirionment.

```sh
cd terraform-docker-demo
terraform init  # Intialize a Terraform working directory, downloading the appropriate provider extensions
terraform plan  # Generate a plan to providsion the described resources.
terraform apply # Apply the plan, provisioning resources.
# App is hosted at localhost:8000.
docker ps # Can see that docker container has been provisioned by Terraform
terraform show # Inspect the current state (Terraform's map of the infrastructure it has created)
terraform destroy # Destroy Terraform created infrastructure.
```

If you look in your current working directory, you'll see that Terraform also wrote some data into the terraform.tfstate file. This state file is extremely important; it keeps track of Terraform's understanding of the resources it created. **We recommended that you use source control for the configuration files, but the state file should not be stored in source control.** You can also setup Terraform Cloud to store and share the state with your teams.

