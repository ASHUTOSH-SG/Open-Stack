



## Setting Up OpenStack


![instance page](https://github.com/user-attachments/assets/25e4a3d3-3703-435e-b866-a91f9a37d17d)


##### A) MicroStack   ---------------------------------------------



---

### 1. System Prep & User Permissions

Checked system info:

```bash
apt install neofetch -y
neofetch
```

Add user (`ashutosh`) to sudo groups for admin access:

```bash
usermod -aG sudo ashutosh
usermod -aG wheel ashutosh
```


```bash
nano /etc/sudoers
```

---

### 2. Install Snap 

Installed Snap package manager:

```bash
sudo apt install snapd -y
```

### MicroStack
Then installed MicroStack (try different versions to avoid channel errors):

```bash
sudo snap install microstack --devmode --classic
sudo snap install microstack --classic
sudo snap install microstack --beta
```

works always
```
sudo snap install microstack --beta
```

---

### 3. Initialize MicroStack

Started the setup with default settings:

```bash
sudo microstack init --auto
```

used control node setup (for single-node use):

```bash
sudo microstack init --auto --control
```

---

### 4. Network & Access Configuration

MicroStack handles networking internally, but to use OpenStack commands, I sourced the RC file:

```bash
source /var/snap/microstack/common/etc/microstack.rc
```

This loads the OpenStack credentials for CLI access.
ID and pass

---

### 5. Test the Deployment

To confirm everything was working:

```
https://10.20.20.1/project
```

```bash
openstack hypervisor list
openstack network list
```

---

### B) devstak  -------------------------------------------

### 1. System Prep & User Setup

Check system info:
```bash
sudo apt install neofetch -y
neofetch
```

Create a non-root user (e.g., `stack`) with sudo privileges:
```bash
sudo adduser stack
sudo usermod -aG sudo stack
```

Switch to the new user:
```bash
su - stack
```

---

### 2. Install Dependencies

Install required system packages:
```bash
sudo apt update
sudo apt install git vim net-tools python3-pip -y
```

---

### 3. Clone DevStack

Clone the DevStack repo (pick a stable branch, e.g., `zed`, `2023.1`, or latest):
```bash
git clone https://opendev.org/openstack/devstack
cd devstack
```

---

### 4. Configure DevStack

Create the `local.conf` file:
```bash
nano local.conf
```

Paste this (minimal setup for single-node all-in-one dev environment):
```ini
[[local|localrc]]
ADMIN_PASSWORD=secret
DATABASE_PASSWORD=$ADMIN_PASSWORD
RABBIT_PASSWORD=$ADMIN_PASSWORD
SERVICE_PASSWORD=$ADMIN_PASSWORD
HOST_IP=127.0.0.1
RECLONE=False
```

Save and exit.

---

### 5. Start DevStack Installation

Run the stack script (this will take some time):
```bash
./stack.sh
```

 If it fails, check the logs at `~/devstack/logs/` or `screen -r` sessions.

---

### 6.Access OpenStack

Once done:

Web Dashboard:
```
http://127.0.0.1/dashboard
```
Login:
- **Username:** `admin`
- **Password:** `secret` 
Source the credentials:
```bash
source openrc admin admin
```

---

### 7. Test the Setup

OpenStack CLI commands to verify:
```bash
openstack compute service list
openstack network list
openstack image list
```

---

### To Re-Run or Reset DevStack

Stop services:
```bash
./unstack.sh
```

Clean up:
```bash
./clean.sh
```

Re-run:
```bash
./stack.sh
```

---

