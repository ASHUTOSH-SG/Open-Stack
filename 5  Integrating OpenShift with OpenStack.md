### Integrating OpenShift with OpenStack  

#### **Requirements**  
1. **OpenStack Implementation** – Needed to deploy OpenShift.  
2. **Domain** – Required for DNS records (e.g., `api.okd.example.com`).  
3. **Linux Installation** – Acts as the admin workstation for CLI operations.  
4. **Python (v3.6+)** – Required for OpenStack CLI tools.  
5. **Two Key Files** – `RC file` & `YAML` (downloaded from OpenStack).  

#### **Steps**  
1. **Create a Project in OpenStack**  
   - Navigate to **Identity → Projects → Create Project** (e.g., `openshift`).  
   - Assign a dedicated user (e.g., `j`) with admin privileges.  

2. **Set Up OpenStack CLI Tools**  
   - Download `RC` and `YAML` files from OpenStack (**Project → API Access**).  
   - Move `clouds.yaml` to `~/.config/openstack/` and secure permissions (`chmod 600`).  
   - Install Python & create a virtual environment:  
     ```bash
     python3 -m venv ~/venv  
     source ~/venv/bin/activate  
     pip install --upgrade pip  
     pip install python-openstackclient  
     ```  
   - Source the `RC` file:  
     ```bash
     source ~/openshift-openrc.sh  
     ```  

3. **Configure OpenStack for OpenShift**  
   - Remove quotas (CPU/RAM) for the project:  
     ```bash
     openstack quota set --cores -1 --ram -1 openshift  
     ```  
   - Create an SSH key (`ssh-keygen -t ed25519 -f ~/.ssh/id_okd`).  
   - Allocate **two Floating IPs** (for API & apps):  
     ```bash
     openstack floating ip create --description "API" <external-network>  
     openstack floating ip create --description "apps" <external-network>  
     ```  

4. **Set Up DNS Records**  
   - Add A records for:  
     - `api.okd.<domain>` → API Floating IP  
     - `*.apps.okd.<domain>` → Apps Floating IP  
   - Verify propagation with `nslookup`.  

5. **Install OpenShift**  
   - Download the OpenShift installer:  
     ```bash
     curl -L -o openshift-install-linux.tar.gz <openshift-installer-url>  
     tar xvf openshift-install-linux.tar.gz  
     ```  
   - Generate manifests & config:  
     ```bash
     ./openshift-install create manifests --dir ~/okd/install  
     ./openshift-install create install-config --dir ~/okd/install  
     ```  
   - Edit `install-config.yaml`:  
     - Add `additionalSecurityGroupIDs: ["<security-group-id>"]` under `controlPlane`.  
     - Add `ingressFloatingIP: "<apps-ip>"` under `platform`.  
   - Run the installer:  
     ```bash
     ./openshift-install create cluster --dir ~/okd/install --log-level info  
     ```  
   - Wait ~30-40 mins for completion.  

6. **Access OpenShift**  
   - Login via the provided URL (`https://api.okd.<domain>:6443`).  
   - Credentials:  
     - Username: `kubeadmin`  
     - Password: (printed in terminal after install).  

#### **Key Takeaways**  
- OpenShift automates VM creation in OpenStack (check **Compute → Instances**).  
- CLI tools (`openstack`, `openshift-install`) streamline deployment.  
- DNS and Floating IPs are critical for routing traffic.  
