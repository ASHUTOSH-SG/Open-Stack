###  Automating OpenStack with Terraform & Ansible**  

#### **Introduction**  
- **Goal**: Automate OpenStack resource provisioning using **Terraform** (infrastructure) and **Ansible** (configuration).  
- **Tools**:  
  - **Terraform**: Creates cloud resources (instances, security groups, etc.).  
  - **Ansible**: Configures servers (installs packages, manages services, etc.).  
- **Series Finale**: This is the last episode of the OpenStack series (sponsored by **OpenMetal**).  

---

### **Example 1: Basic Terraform Setup**  
1. **Install Terraform**:  
   - Download the Linux binary from [terraform.io](https://terraform.io).  
   - Unzip and move to `/usr/local/bin`:  
     ```bash
     wget <terraform-url>  
     unzip terraform*.zip  
     sudo mv terraform /usr/local/bin/  
     ```  
   - Verify: `terraform --version`.  

2. **Create a Project in OpenStack**:  
   - **Identity → Projects → Create Project** (e.g., `terraform-sandbox`).  
   - Assign an admin user (e.g., `j`).  

3. **Generate SSH Key Pair**:  
   - **Compute → Key Pairs → Create Key Pair** (e.g., `terraform-testing`).  
   - Save the private key in `~/.ssh/`.  

4. **Terraform Configuration**:  
   - Create `example.tf`:  
     ```hcl
     terraform {
       required_providers {
         openstack = {
           source = "terraform-provider-openstack/openstack"
         }
       }
     }

     provider "openstack" {
       username   = "j"
       tenant_name = "terraform-sandbox"
       password   = "your-password"
       auth_url   = "http://<openstack-ip>:5000/v3"
       region     = "iad3"
       insecure   = true  # For HTTP (no SSL)
     }

     resource "openstack_compute_instance_v2" "webserver" {
       name            = "webserver"
       image_id        = "<image-id>"
       flavor_id       = "<flavor-id>"
       key_pair        = "terraform-testing"
       security_groups = ["default"]
       network {
         name = "external"
       }
     }
     ```  
   - Initialize: `terraform init`.  
   - Plan: `terraform plan`.  
   - Apply: `terraform apply` → Confirms instance creation.  
   - Destroy: `terraform destroy` → Removes resources.  

---

### **Example 2: Splitting Terraform Files**  
- **Organization**: Split into `provider.tf` (provider config) and `compute.tf` (instance config).  
- **Key Takeaway**: Terraform automatically combines all `.tf` files in a directory.  

---

### **Example 3: Using Variables**  
- **Problem**: Hardcoding secrets (e.g., passwords) is insecure.  
- **Solution**: Use `variables.tf`:  
  ```hcl
  variable "openstack_password" {
    type    = string
    default = "your-password"
  }
  ```  
- **Update `provider.tf`**: Replace hardcoded values with `var.openstack_password`.  
- **Best Practice**: Keep `variables.tf` **out of version control** (e.g., Git).  

---

### **Example 4: Adding Security Groups**  
- **New File**: `security_group.tf`:  
  ```hcl
  resource "openstack_compute_secgroup_v2" "webserver" {
    name        = "webserver-sg"
    description = "Allow SSH/HTTP"
    rule {
      ip_protocol = "tcp"
      from_port   = 22
      to_port     = 22
      cidr        = var.admin_ip  # Restrict to your IP
    }
    rule {
      ip_protocol = "tcp"
      from_port   = 80
      to_port     = 80
      cidr        = "0.0.0.0/0"  # Allow HTTP globally
    }
  }
  ```  
- **Update `compute.tf`**: Reference the new SG (`security_groups = ["webserver-sg"]`).  

---

### **Example 5: Integrating Ansible**  
1. **Install Ansible**:  
   ```bash
   sudo apt install ansible
   ```  

2. **Terraform Provisioner**:  
   - Add to `compute.tf`:  
     ```hcl
     provisioner "local-exec" {
       command = "sleep 45 && ansible-playbook -u ubuntu -i '${self.access_ip_v4},' --private-key ~/.ssh/terraform-testing apache.yml"
     }
     ```  

3. **Ansible Playbook (`apache.yml`)**:  
   ```yaml
   - name: Install Apache
     hosts: all
     become: yes
     tasks:
       - name: Update packages
         apt:
           update_cache: yes
           upgrade: dist
       - name: Install Apache
         apt:
           name: apache2
           state: latest
   ```  

4. **Run**:  
   - `terraform apply` → Creates instance, runs Ansible to install Apache.  
   - Verify: Access the instance IP in a browser → Apache default page.  

---

### **Final Notes**  
- **Terraform**: Manages **infrastructure as code** (IaC).  
- **Ansible**: Handles **post-deployment configuration**.  
- **OpenMetal**: Simplifies OpenStack deployment (sponsor).  

