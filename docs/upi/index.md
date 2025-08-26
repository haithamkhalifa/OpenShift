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
   - Create VMs or physical hosts (Deployer(Bastion),Bootstrap, Control Plane, Workers)
   - Allocate IPs, MAC addresses, VLANs
   - Configure DNS (api + *.apps records)
   - Configure Load Balancer (HAProxy, F5, Nginx, etc.)
   - Configure DHCP (or static IPs)

2. **Generate installation artifacts**
   - Create `install-config.yaml`
   - Run `openshift-install create manifests`
   - Run `openshift-install create ignition-configs`

3. **Provision bootstrap + masters**
   - Attach ignition files (via HTTP/pxe/iso)
   - Start bootstrap node
   - Start master nodes

4. **Wait for bootstrap complete**
   ```bash
   openshift-install wait-for bootstrap-complete

5. **Join worker nodes**
   ```bash
   oc get csr
   oc adm certificate approve <csr-name>
6. **Verify cluster**
   ```bash
   oc get nodes
   oc get co
   
