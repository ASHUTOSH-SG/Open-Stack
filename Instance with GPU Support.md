
# Ubuntu Instance with GPU Support

## Launching Instance via Horizon Dashboard

 **Login**  
   - Open browser: `http://<controller_ip>/dashboard`

 **Go to**  
   - Project → Compute → Instances → Launch Instance

 **Details Tab**
   - Instance Name: `ubuntu-instance`
   - Availability Zone: default
   - Count: `1`

 **Source Tab**
   - Select Ubuntu image
   - Create New Volume: `Yes` (optional)
   - Volume Size: e.g., `20 GB`
   - Boot from: `Image (creates new volume)`

**Flavor Tab**
   - Select existing flavor (e.g., `m1.small`, `m1.medium`)
   - Or create new one via: Admin → Compute → Flavors

**Networks Tab**
   - Select network (e.g., `private` or `provider`)

**Security Groups Tab**
   - Add `default` group
   - Or create one with:
     - SSH (port 22)
     - ICMP (ping)
     - RDP (port 3389) for GUI

**Key Pair Tab**
   - Select existing key pair
   - Or import a new one

**Click Launch**

---

## GPU Passthrough Setup (Using KVM/QEMU)

> VirtualBox does **not** support GPU passthrough  
> Use KVM/QEMU on a bare-metal machine with **Intel VT-d** or **AMD-Vi**

## Does OpenStack support GPU passthrough ?
Yes, but only if:

You're using KVM/QEMU as the hypervisor (not VirtualBox).

The host machine is bare-metal (not itself a VM).

You have Intel VT-d or AMD-Vi enabled in BIOS.

You configure IOMMU, nova.conf, and create GPU flavors (as in the steps you just noted).


KVM is a type 1 hypervisor, which essentially means it is able to run on bare metal. QEMU is a type 2 hypervisor, which means that it runs on top of the operating system. In this case, QEMU will utilize KVM in order to utilize the machine’s physical resources for the virtual machines.





---

**Step 1**  
**Action**: Enable VT-d (for Intel) or AMD-Vi (for AMD) in BIOS  
**Full Form**:  
- VT-d = Intel Virtualization Technology for Directed I/O  
- AMD-Vi = AMD Virtualization for I/O  
**Purpose**: This enables **IOMMU** (Input-Output Memory Management Unit) support, which is necessary for GPU passthrough.

---

**Step 2**  
**Command**:  
```bash
dmesg | grep -e DMAR -e IOMMU
```  
**Full Form**:  
- DMAR = DMA Remapping (Intel’s implementation of VT-d)  
- IOMMU = Input-Output Memory Management Unit  
**Purpose**: Verifies whether IOMMU is active and recognized by the kernel after booting.

---

**Step 3**  
**Action**: Add a kernel parameter in GRUB  
- For Intel: `intel_iommu=on`  
- For AMD: `amd_iommu=on`  
**Full Form**:  
- GRUB = Grand Unified Bootloader  
**Purpose**: This ensures IOMMU is enabled at boot time.  
**Note**: After editing `/etc/default/grub`, run `update-grub` and reboot.

---

**Step 4**  
**Command**:  
```bash
lspci -nn
```  
**Full Form**:  
- lspci = List PCI (Peripheral Component Interconnect) devices  
**Purpose**: Lists all PCI devices in your system so you can identify your GPU by its **vendor and device ID**.

---

**Step 5**  
**Command**:  
```bash
find /sys/kernel/iommu_groups/ -type l
```  
**Full Form**:  
- sysfs = Virtual file system showing kernel objects  
**Purpose**: Verifies if your GPU is in a separate IOMMU group, which is required for **safe and isolated passthrough**.

---

**Step 6**  
**Action**: Install NVIDIA GPU drivers on the **host** machine  
**Full Form**:  
- NVIDIA = Company that makes GPUs (e.g., RTX, Quadro, Tesla)  
**Purpose**: Ensures that the host system can manage the GPU and pass it properly to VMs.

---

**Step 7**  
**Action**: Edit the file `/etc/nova/nova.conf`  
- Add this under `[pci]` section:  
```ini
[pci]
passthrough_whitelist = {"vendor_id":"10de","product_id":"1db6"}
alias = {"name":"gpu","product_id":"1db6","vendor_id":"10de","device_type":"type-PCI"}
```  
**Full Form**:  
- Nova = OpenStack Compute service  
**Purpose**: This tells Nova that your GPU (by vendor and product ID) is available for passthrough to VMs.

---

**Step 8**  
**Command**:  
```bash
systemctl restart nova-compute
```  
 
**Purpose**: Applies the changes you made in the nova configuration.

---

**Step 9**  
**Command**:  
```bash
openstack flavor create gpu.flavor --ram 4096 --disk 40 --vcpus 2 --property "pci_passthrough:alias"="gpu:1"
```  
**Full Form**:  
- Flavor = Template for VM hardware configuration (RAM, disk, CPU, GPU, etc.)  
**Purpose**: Creates a special flavor that requires **1 GPU** when the VM is launched.

---

**Step 10**  
**Command**:  
```bash
openstack server create --flavor gpu.flavor --image ubuntu-image --network private-net --key-name mykey ubuntu-gpu-instance
```  
**Full Form**:  
- Server = Virtual Machine = Instance  
**Purpose**: Launches a new Ubuntu virtual machine using the GPU flavor, which should attach the GPU to the instance.

---

**Final Step – Test Inside the VM**  
After logging into the VM, run:  
```bash
nvidia-smi
```  
