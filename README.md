## 🔑 Keycloak on Google Cloud Platform (GCP) Deployment
<!--
  Copyright 2025 Google LLC

  Licensed under the Apache License, Version 2.0 (the "License");
  you may not use this file except in compliance with the License.
  You may obtain a copy of the License at

       http://www.apache.org/licenses/LICENSE-2.0

  Unless required by applicable law or agreed to in writing, software
  distributed under the License is distributed on an "AS IS" BASIS,
  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  See the License for the specific language governing permissions and
  limitations under the License.
-->

This repository contains the Terraform configuration files necessary to deploy an instance of **Keycloak** (an open-source Identity and Access Management solution) on Google Cloud Platform (GCP).

### ✨ What is Deployed?

This configuration deploys Keycloak on a single **Compute Engine VM** and places it behind a **Global External HTTPS Load Balancer** for secure, public access.

The main components provisioned include:

* **VPC Router and NAT Gateway:** Enables the Keycloak VM (which uses an internal IP) to access the internet.
* **Compute Engine Instance:** An **`e2-medium`** VM running Ubuntu, which executes a startup script to download, install and run Keycloak.
* **Load Balancing Components:**
    * **Global External IP Address:** The public entry point.
    * **Managed SSL Certificate:** Automatically provisioned by Google to provide **HTTPS** (port 443) access using a domain name derived from the IP address (via `sslip.io` and `nip.io`).
    * **Backend Service, NEG, and Health Checks:** Configured to direct traffic to the Keycloak VM on port `8080`.
* **Firewall Rule:** Allows traffic from the Load Balancer health check range to the VM on port `8080`.

---

## 🛠️ Prerequisites

Before running this deployment, ensure you have the following installed and configured:

1.  **Terraform CLI:** Installed on your local machine.
2.  **Google Cloud CLI (gcloud):** Installed and authenticated.
    * Ensure you have run `gcloud auth application-default login` to grant Terraform the necessary permissions to manage resources in your GCP project.
3.  **Repository Contents:** Clone this repo.

---

## 🚀 Deployment Instructions

### 1. Define Variables (Using Environment Variables)

This configuration requires several input variables. For secure and non-interactive deployment, it is recommended to pass these values using **`TF_VAR_` environment variables**.

| Variable Name                    | Description                                           | Example Value             |
|:---------------------------------|:------------------------------------------------------|:--------------------------|
| `TF_VAR_gcp_project_id`          | Your GCP Project ID                                   | `my-secure-project-12345` |
| `TF_VAR_gcp_region`              | Primary GCP Region                                    | `us-central1`             |
| `TF_VAR_gcp_zone`                | Primary GCP Zone                                      | `us-central1-a`           |
| `TF_VAR_keycloak_admin`          | Keycloak Initial Admin Username                       | `admin`                   |
| `TF_VAR_keycloak_admin_password` | Keycloak Initial Admin Password (Secret)              | `MySecurePassword123`     |
| `TF_VAR_vpc_name`                | Name of the VPC to use (defaults to `default`)        | `my-vpc`                  |
| `TF_VAR_vpc_subnet`              | Name of the VPC subnet to use (defaults to `default`) | `my-vpc-subnet`           |


**Example of setting environment variables (Linux/macOS):**

```bash
export TF_VAR_gcp_project_id="<YOUR_PROJECT_ID>"
export TF_VAR_gcp_region="us-central1"
export TF_VAR_gcp_zone="us-central1-a"
export TF_VAR_keycloak_admin="admin"
export TF_VAR_keycloak_admin_password="MySupeSecretPass123!"
```

### 2. Initialize Terraform

Run `terraform init` to download the required GCP provider and set up the working directory. This command only needs to be run once per deployment directory.

```bash
terraform init
```

### 3. Apply the Configuration

Run `terraform apply` to create the infrastructure defined in the `.tf` files. The deployment will automatically use the environment variables you set in Step 1.

```bash
# Optional: Preview the changes
terraform plan

# Apply the changes and provision resources
# Add -auto-approve flag to skip confirmation prompt
terraform apply -auto-approve
```

### 4. Access Keycloak

After a successful deployment, the Keycloak instance will be accessible via the Load Balancer's public IP address.

To get the final access URL, run `terraform output`:

```bash
# This assumes an output block for the URL has been defined in your configuration
terraform output keycloak-url
```

The output will provide the final URL, typically formatted as: `https://<EXTERNAL_IP_ADDRESS>.nip.io/`. Allow for about 15 minutes for the managed SSL certificate to become active.

You can log in to the **Keycloak Admin Console** using the admin credentials you set via the environment variables.

---

## 🗑️ Cleanup

To destroy all provisioned resources and avoid incurring charges, run the following command. **Warning:** This action is permanent and deletes your entire Keycloak deployment.

```bash
terraform destroy
```


### Not Google Product Clause

This is not an officially supported Google product.