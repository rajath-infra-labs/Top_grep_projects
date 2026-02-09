Terraform GCP VM Lab - README
Purpose

This lab demonstrates how to create a small Google Cloud VM using Terraform, troubleshoot common errors, and understand authentication, APIs, and billing setup.

Steps We Followed
1. Set up GCP

Created a new project:

gcloud projects create terraform-rrao-asia-2026 --name="Terraform Lab"


Set the project:

gcloud config set project terraform-rrao-asia-2026


Enabled billing for the project (required to use Compute Engine):

Done via GCP Console → Billing → Link Billing Account

2. Set up Authentication

Logged in to GCP CLI:

gcloud auth login


Configured Application Default Credentials (ADC) for Terraform:

gcloud auth application-default login


Error we saw: No credentials loaded → fixed using ADC login.

3. Enable Required APIs

Compute Engine API must be enabled to create VMs:

gcloud services enable compute.googleapis.com --project=terraform-rrao-asia-2026


Error we saw: Billing must be enabled → fixed by linking billing.

4. Terraform Setup

Created project files:

terraform-vm/
  ├─ main.tf
  ├─ variables.tf (optional)
  ├─ outputs.tf (optional)


main.tf example:

provider "google" {
  project = var.project_id
  region  = var.region
  zone    = var.zone
}

resource "google_compute_instance" "vm" {
  name         = "tf-vm-1"
  machine_type = "e2-micro"
  zone         = var.zone

  boot_disk {
    initialize_params { image = "debian-cloud/debian-11" }
  }

  network_interface {
    network = "default"
    access_config {} # Assigns public IP
  }
}

5. Run Terraform

Initialize Terraform:

terraform init


Plan resources:

terraform plan


Apply changes:

terraform apply


Type yes when prompted.

VM created successfully in zone asia-south1-a.

6. Accessing the VM

SSH into VM from terminal:

gcloud compute ssh tf-vm-1 --zone asia-south1-a


Debian 11 OS is installed by default:

Linux-based OS

CLI available

Can run commands like:

ls
pwd
sudo apt update

7. Clean-up

Destroy resources after testing:

terraform destroy


Prevents unnecessary billing.

Common Errors Encountered
Error	Cause	Fix
No credentials loaded	Terraform couldn’t find ADC	gcloud auth application-default login
Compute Engine API not enabled	API not activated	gcloud services enable compute.googleapis.com
Billing account not found	Project had no billing	Link billing account in GCP Console
Project ID already in use	GCP requires unique IDs	Pick a unique project ID
Key Learnings

Terraform requires Application Default Credentials to work with GCP.

Billing must be enabled before enabling APIs or creating resources.

Terraform does not automatically enable APIs.

access_config {} in network interface gives public IP to VM.

debian-cloud/debian-11 is the OS running on the VM.

Always run terraform destroy to avoid extra costs.
