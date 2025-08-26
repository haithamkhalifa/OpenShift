# OpenShift Installation — UPI (User-Provisioned Infrastructure)

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
  - DNS server → `dnsmasq`
  - DHCP server → `dnsmasq`
  - Load balancer → `haproxy`
  - Private image registry → **Red Hat Quay**
      ```
      # https://docs.redhat.com/en/documentation/openshift_container_platform/4.18/html-single/disconnected_environments/index#installing-mirroring-creating-registry
      curl -LO https://mirror.openshift.com/pub/cgw/mirror-registry/latest/mirror-registry-amd64.tar.gz
      ### INSTALL Quay ###	
      ./mirror-registry install \
        --quayHostname quay.openshifty.duckdns.org \
        --quayRoot /registry/quayRoot \
        --initPassword P@ssw0rd \
        --initUser devops \
        --quayStorage /registry/quayStorage \
        --sqliteStorage /registry/sqliteStorage \
        --ssh-key /home/devops/.ssh/id_rsa_quay \
        --sslCert /registry/quayRoot/quay-config/fullchain.crt \
        --sslKey /registry/quayRoot/quay-config/openshifty.duckdns.org.key \
        --targetHostname quay.openshifty.duckdns.org \
        --targetUsername devops \
        --verbose

  - download and install needed tools
      ```bash 
      curl -LO https://mirror.openshift.com/pub/openshift-v4/clients/ocp/4.18.21/oc-mirror.tar.gz
      tar -xvzf oc-mirror.tar.gz
      sudo mv oc-mirror /usr/local/bin/
      mkdir ~/.docker/
      cat ~/pull-secret.txt | jq . > ~/.docker/config.json #get pull-secret from [Red Hat Console](https://console.redhat.com/openshift/downloads)
- Provision VMs or physical hosts:
  - **Bootstrap**
  - **Control Plane (Masters)**
  - **Workers**
- Allocate IPs, MAC addresses, VLANs  
- Configure **DNS** (`api` + `*.apps` records)  
- Configure **Load Balancer** (HAProxy, F5, Nginx, etc.)  
- Configure **DHCP** (or assign static IPs)   

### 2. Generate Installation Artifacts
- **Mirror OpenShift Release Images**
   *ImageSetConfiguration.yaml
   ```bash
   cat <<'EOF' >ImageSetConfiguration.yaml
      kind: ImageSetConfiguration
      apiVersion: mirror.openshift.io/v2alpha1
      mirror:
        platform:
          architectures:
            - "amd64"
          channels:
          - name: stable-4.18 
            minVersion: 4.18.1
            maxVersion: 4.18.1
          graph: true
        operators:
          - catalog: registry.redhat.io/redhat/redhat-operator-index:v4.18
            packages:
              - name: file-integrity-operator
                channels:
                  - name: stable
              - name: cluster-logging
                channels:
        additionalImages: 
        additionalImages: 
         - name: registry.redhat.io/ubi8/ubi:latest
         - name: registry.redhat.io/ubi9/ubi@sha256:20f695d2a91352d4eaa25107535126727b5945bff38ed36a3e59590f495046f0
      EOF
   # 1. Mirror to disk: export the image set into an archive
   nohup oc mirror -c ./ImageSetConfiguration.yaml file://./ --v2 > oc-mirror-to-disk.out &
   # 2. Transfer the archive to the disconnected network manually
   # 3. Disk to mirror: import the archive into your disconnected registry
   nohup oc mirror -c ./ImageSetConfiguration.yaml \
   --from file://./ docker://quay.openshifty.duckdns.org:8443 --v2 \
   > oc-disk-to-mirror.out & 

### 3. **Generate installation artifacts**
   - Mirror OpenShift Relesae Images
	```bash
	#1. Mirror to disk: Mirror the image set to an archive.
	#oc mirror -c <image_set_configuration> file://<file_path> --v2
	nohup oc mirror -c ./ImageSetConfiguration.yaml file://./ --v2 > oc-mirror-to-disk.out &
	#2. Disk transfer: Manually transfer the archive to the network of the disconnected mirror registry.
	#3. Disk to mirror: Mirror the image set from the archive to the target disconnected registry.
	#oc mirror -c <image_set_configuration> --from file://<file_path> docker://<mirror_registry_url> --v2
	nohup oc mirror -c ./ImageSetConfiguration.yaml --from file://./ docker://quay.openshifty.duckdns.org:8443 --v2 > oc-disk-to-mirror.out &
   - Create `install-config.yaml`
   - Run `openshift-install create manifests`
   - Run `openshift-install create ignition-configs`

### 5. **Provision bootstrap + masters**
   - Attach ignition files (via HTTP/pxe/iso)
   - Start bootstrap node
   - Start master nodes

### 6. **Wait for bootstrap complete**
   ```bash
   openshift-install wait-for bootstrap-complete

### 7. **Join worker nodes**
   ```bash
   oc get csr
   oc adm certificate approve <csr-name>
### 8. **Verify cluster**
   ```bash
   oc get nodes
   oc get co
   
