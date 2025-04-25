###  OpenStack Compute (Nova) 

#### **1. Introduction**  
- **OpenStack Horizon**: Dashboard for managing OpenStack.  
- **Compute (Nova)**: Core component responsible for provisioning and managing virtual machines (instances).  
- **Previous Episode**: Covered Horizon administration.  
- **This Episode**: Focuses on creating compute instances, snapshots, and managing flavors.  

---  

#### **2. Creating a Compute Instance**  
1. **Log in to Horizon Dashboard**  
   - Username & password authentication.  

2. **Launch Instance**  
   - **Name & Description**: Use a structured naming scheme (e.g., `blog-prod-1`).  
   - **Availability Zone & Count**: Default settings (usually `nova` and `1`).  

3. **Source (Image Selection)**  
   - Choose a base OS image (e.g., Ubuntu 20.04).  
   - **Volume Size**: Set disk size (e.g., 16GB).  
   - **Delete Volume on Instance Delete**:  
     - **No**: Retains disk after instance deletion (good for data recovery).  
     - **Yes**: Deletes disk along with instance.  

4. **Flavor (Instance Type)**  
   - Determines CPU, RAM, and disk resources.  
   - Example: `gp1.micro` (1 vCPU, 1GB RAM, 25GB disk).  

5. **Networking**  
   - Assign to a network (default: `external`).  

6. **Security Groups**  
   - Acts as a firewall.  
   - Default group allows basic traffic.  
   - Add `allow-ssh-ingress` to enable SSH access (port 22).  

7. **Key Pair (SSH Authentication)**  
   - Generate a new key pair (e.g., `j_key.pem`).  
   - Save the private key securely (`~/.ssh/` with `chmod 600`).  

8. **Launch Instance**  
   - Verify in the **Instances** tab.  
   - Access via **Console** or **SSH** (e.g., `ssh -i ~/.ssh/j_key.pem ubuntu@<IP>`).  

---  

#### **3. Post-Instance Setup**  
- **Update Packages**:  
  ```bash
  sudo apt update && sudo apt dist-upgrade -y
  ```  
- **Install Software**:  
  ```bash
  sudo apt install apache2
  ```  
- **Security Groups for Web Access**:  
  - Add `HTTP` rule (port 80) to allow web traffic.  

---  

#### **4. Snapshots**  
- **Purpose**: Backup an instance before changes (e.g., testing updates).  
- **Create Snapshot**:  
  - From **Instances** → **Create Snapshot** (e.g., `test-snapshot`).  
  - Stored under **Images**.  
- **Restore Snapshot**:  
  - Launch a new instance from the snapshot.  
  - Retains original OS and configurations.  

---  

#### **5. Flavors (Instance Types)**  
- **Definition**: Predefined resource templates (vCPUs, RAM, disk).  
- **Examples**:  
  - `gp1.micro`: General-purpose, low resources.  
  - `c1.medium`: Compute-optimized.  
- **Custom Flavors**:  
  - **Admin** → **Compute** → **Flavors** → **Create Flavor**.  
  - Specify vCPUs, RAM, disk (e.g., `gp.custom1`: 4 vCPUs, 4GB RAM).  

---  

#### **6. Resizing Instances**  
- **When to Resize**: Resource contention (e.g., CPU/RAM limits).  
- **Process**:  
  - **Instances** → **Resize** → Choose new flavor.  
  - **Note**: Instance reboots during resize (downtime).  

---  

#### **7. Server Groups**  
- **Purpose**: Control instance placement on Nova nodes.  
- **Policies**:  
  - **Affinity**: Run instances on the same node.  
  - **Anti-Affinity**: Run instances on different nodes (e.g., for redundancy).  
- **Soft Policies**: Uses weights for automatic placement.  

---  

#### **8. Key Takeaways**  
- **Nova**: Manages compute resources (VMs).  
- **Instances**: Created from images, customized with flavors.  
- **Snapshots**: Backup/restore points.  
- **Security Groups**: Firewall rules for network access.  
- **Server Groups**: High-availability strategies.  

