### OpenStack Networking (Neutron) 

#### **1. Introduction**  
  - **Public vs. Private Networks**:  
    - **Public**: Accessible from the internet (e.g., web servers).  
    - **Private**: Internal-only (e.g., management networks).  
  - **Security Groups**: Act as firewalls for instances.  
  - **Floating IPs**: Dynamic public IPs that can be reassigned.  
  - **Load Balancers**: Distribute traffic across instances.  

---  

#### **2. Security Groups (Firewall Rules)**  
- **Purpose**: Control inbound/outbound traffic to instances.  
- **Example**:  
  - **SSH Access**: Restrict to a specific IP (e.g., `192.168.1.100/32`).  
  - **Web Server**: Allow HTTP (port `80`) and HTTPS (port `443`).  

##### **Creating a Security Group**  
1. **Navigate**: `Network` → `Security Groups` → `Create Security Group`.  
2. **Name**: `ssh-access` (or descriptive name).  
3. **Add Rule**:  
   - **Rule**: `SSH` (port `22`).  
   - **Remote IP**: Your office IP (e.g., `192.168.1.100/32`).  

##### **Attaching to Instances**  
- **Steps**:  
  - Go to `Compute` → `Instances`.  
  - Click `Edit Security Groups` on an instance.  
  - Attach the `ssh-access` group.  

---  

#### **3. Private Networks & Routers**  
##### **Creating a Private Network**  
1. **Navigate**: `Network` → `Networks` → `Create Network`.  
   - **Name**: `private-network-1`.  
   - **Subnet**: `192.168.0.0/24` (private IP range).  
   - **Gateway IP**: Leave blank (Neutron auto-assigns).  

##### **Creating a Router**  
1. **Navigate**: `Network` → `Routers` → `Create Router`.  
   - **Name**: `private-router-1`.  
   - **External Network**: Choose the default provider network (e.g., `external`).  
2. **Add Interface**:  
   - Attach to `private-network-1`.  

---  

#### **4. Floating IPs (Dynamic Public IPs)**  
- **Use Case**: Assign a public IP to a private instance.  
- **Steps**:  
  1. **Allocate IP**:  
     - `Network` → `Floating IPs` → `Allocate IP`.  
  2. **Associate to Instance**:  
     - From `Compute` → `Instances`, click `Associate Floating IP`.  
  3. **Reassigning**:  
     - Disassociate from one instance → Associate to another.  

---  

#### **5. Load Balancers**  
- **Purpose**: Distribute traffic across multiple instances.  
- **Example Setup**:  
  1. **Create Load Balancer**:  
     - `Network` → `Load Balancers` → `Create Load Balancer`.  
     - **Name**: `web-lb`.  
     - **Subnet**: `private-network-1`.  
     - **Protocol**: `HTTP` (port `80`).  
     - **Algorithm**: `Round Robin` (even distribution).  
  2. **Add Instances**:  
     - Select instances to balance (e.g., `instance-1`, `instance-2`).  
  3. **Attach Floating IP**:  
     - Assign a floating IP to the load balancer (not instances).  

##### **Testing the Load Balancer**  
- Access the floating IP in a browser.  
- Refresh repeatedly → Traffic alternates between instances.  

---  

#### **6. Key Takeaways**  
- **Security Groups**: Restrict access (e.g., SSH only from trusted IPs).  
- **Private Networks**: Isolate internal services (e.g., databases).  
- **Floating IPs**: Enable redundancy (failover between instances).  
- **Load Balancers**: Improve scalability & availability.  

---  
