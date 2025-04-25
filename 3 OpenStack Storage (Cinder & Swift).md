### OpenStack Storage (Cinder & Swift)
#### **1. Introduction**  

- **Key Services**:  
  - **Cinder**: Block storage (like virtual hard disks).  
  - **Swift**: Object storage (for scalable, unstructured data).  

---  

#### **2. Block Storage with Cinder**  
- **Volumes**: Virtual disks attached to instances.  
  - Created automatically with instances (root volumes).  
  - Can also create **additional volumes** for extra storage.  

##### **Creating & Attaching a Volume**  
1. **Create Volume**:  
   - **Dashboard** → **Volumes** → **Create Volume**.  
   - Name (e.g., `blog-data`), size (e.g., 16GB), type (default).  
   - **Availability Zone**: Must match the instance’s zone (usually `nova`).  

2. **Attach to Instance**:  
   - **Volumes** → **Manage Attachments** → Select instance.  
   - Device path (e.g., `/dev/vdc`).  

##### **Setting Up the Volume in Linux**  
1. **Check Disk**:  
   ```bash
   lsblk  # Lists block devices (e.g., /dev/vdc).
   ```  
2. **Partition & Format**:  
   ```bash
   sudo fdisk /dev/vdc  # Create partition (e.g., /dev/vdc1).
   sudo mkfs.ext4 /dev/vdc1  # Format as ext4.
   ```  
3. **Mount the Volume**:  
   ```bash
   sudo mkdir /mnt/storage1  # Create mount point.
   sudo mount /dev/vdc1 /mnt/storage1  # Mount.
   df -h  # Verify mount.
   ```  

##### **Detaching & Reattaching Volumes**  
- **Detach**:  
  - Unmount first (`sudo umount /mnt/storage1`).  
  - In Horizon: **Volumes** → **Detach**.  
- **Reattach to Another Instance**:  
  - Same process; data persists.  

---  

#### **3. Object Storage with Swift**  
- **Use Case**: Scalable storage for files (e.g., backups, static websites).  
- **Containers (Buckets)**: Hold objects (files) without file systems.  

##### **Creating a Container & Uploading Files**  
1. **Create Container**:  
   - **Dashboard** → **Object Store** → **Containers** → **Create Container**.  
   - Name (e.g., `test_container`), set as **public/private**.  

2. **Upload Files**:  
   - Upload via UI (e.g., `index.html`).  
   - Access via public URL:  
     ```
     http://<openstack-url>/<container-name>/<file-name>
     ```  

---  

#### **4. Key Differences: Block vs. Object Storage**  
| **Feature**       | **Block Storage (Cinder)**       | **Object Storage (Swift)**       |  
|-------------------|----------------------------------|----------------------------------|  
| **Use Case**      | VM disks, databases.             | Static files, backups, web hosting. |  
| **Structure**     | Formatted (ext4, NTFS).          | No file system (key-value pairs). |  
| **Access**        | Attached to one instance at a time. | Accessible via HTTP/API. |  
| **Scalability**   | Fixed size (e.g., 16GB).         | Virtually unlimited. |  

---  

#### **5. Practical Tips**  
- **Retention Policies**: Detach & retain volumes after instance deletion.  
- **Data Transfer**: Move volumes between instances instead of network copies.  
- **Security**: Restrict public access to object storage unless needed.  

---  
