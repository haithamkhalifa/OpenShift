# OpenShift Installation — UPI (User-Provisioned Infrastructure)
## Disconnected (Air-gapped)

The **User-Provisioned Infrastructure (UPI)** method gives you **full control** over the infrastructure layer.  
Unlike **IPI (Installer-Provisioned Infrastructure)**, with UPI you must manually configure:

- Networking  
- Load balancing  
- DNS  
- DHCP  
- Storage  

---

## 🔹 UPI Workflow (High-level)

### 1. Prepare Infrastructure
- Create a **Deployer (Bastion) VM** hosting:
  - **DNS server → `dnsmasq`**
  - **DHCP server → `dnsmasq`**
  - **Load balancer → `haproxy`**
  - **Private image registry → ``Red Hat Quay``**
- Provision VMs or physical hosts:
  - **Bootstrap**
  - **Control Plane (Masters)**
  - **Workers**
- Allocate IPs, MAC addresses, VLANs  
- Configure **DNS** (`api` + `*.apps` records)  
- Configure **Load Balancer** (HAProxy, F5, Nginx, etc.)  
- Configure **DHCP** (or assign static IPs)

### 2. Deployer (Bastion) setup
- Install RHEL8 or RHEL9 as os.
- Disable the FW and SElinux.
- hereunder all needed.
```bash 
   sudo dnf install -y dnsmasq haproxy
