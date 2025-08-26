# OpenShift Installation — UPI (User-Provisioned Infrastructure)

The **User-Provisioned Infrastructure (UPI)** method gives you full control over the infrastructure layer.  
Unlike IPI (Installer-Provisioned Infrastructure), you must manually configure **networking, load balancing, DNS, DHCP, and storage**, then provide ignition files to bootstrap the cluster.

---

## When to use UPI?
- ✅ Bare metal clusters
- ✅ Custom lab environments (Proxmox, VMware, physical servers)
- ✅ Disconnected / air-gapped installations
- ✅ Advanced setups where you control DNS, DHCP, HAProxy, certificates

---

## UPI Workflow (High-level)

1. **Prepare infrastructure**
   - Create Deployer (Bastion) VM
      - it will host the following services:
         - DNS Server   -> dnsmasq
         - DHCP Server  -> dnsmasq
         - LoadBalancer -> haproxy
         - Private Image Registry -> Red Hat Quay     
   - Create VMs or physical hosts (Bootstrap, Control Plane, Workers)
   - Allocate IPs, MAC addresses, VLANs
   - Configure DNS (api + *.apps records)
   - Configure Load Balancer (HAProxy, F5, Nginx, etc.)
   - Configure DHCP (or static IPs)

3. **Generate installation artifacts**
   - Create `install-config.yaml`
   - Run `openshift-install create manifests`
   - Run `openshift-install create ignition-configs`

4. **Provision bootstrap + masters**
   - Attach ignition files (via HTTP/pxe/iso)
   - Start bootstrap node
   - Start master nodes

5. **Wait for bootstrap complete**
   ```bash
   openshift-install wait-for bootstrap-complete

6. **Join worker nodes**
   ```bash
   oc get csr
   oc adm certificate approve <csr-name>
7. **Verify cluster**
   ```bash
   oc get nodes
   oc get co
   
