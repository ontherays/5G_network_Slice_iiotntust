
### QEMU/KVM + UERANSIM Setup for Your Environment**  
**Host Machine**:  
- **OS**: Ubuntu 22.04  
- **Host IP**: `192.168.5.234` (Wi-Fi, `wlp58s0`)  
- **Libvirt NAT IP**: `192.168.122.1`  
- **Open5GS AMF**: Running on host  

**VM (UERANSIM)**:  
- **VM IP**: `192.168.122.234` (NAT)  
- **Goal**: Connect UERANSIM (gNB + UE) in VM to Open5GS (AMF) on host.  

---

## **Step 1: Install QEMU/KVM on Host**  
### **1. Verify Virtualization Support**  
```bash
egrep -c '(vmx|svm)' /proc/cpuinfo  # Must return ≥1
```  
- **If 0**: Enable VT-x/AMD-V in BIOS.  

### **2. Install Packages**  
```bash
sudo apt update
sudo apt install -y qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils virt-manager
```  

### **3. Add User to Groups**  
```bash
sudo usermod -aG libvirt $USER
sudo usermod -aG kvm $USER
```  
- **Log out and back in**.  

---

## **Step 2: Create VM for UERANSIM**  
### **1. Download Ubuntu 22.04 ISO**  
```bash
wget https://releases.ubuntu.com/22.04/ubuntu-22.04.3-live-server-amd64.iso
```  

### **2. Create VM via `virt-manager`**  
- **ISO**: Select downloaded Ubuntu ISO.  
- **Resources**:  
  - CPU: 2+ cores  
  - RAM: 4GB  
  - Storage: 20GB (QCOW2)  
- **Network**:  
  - **NAT (default)**: VM will use `192.168.122.0/24` subnet.  
  - **Bridge (if needed)**: Attach to `br0` (see [previous steps](#step-2-configure-networking)).  

---

## **Step 3: Install UERANSIM in VM**  
### **1. SSH into VM**  
```bash
virsh console <VM_NAME>  # Or use `virt-manager` GUI
```  

### **2. Install Dependencies**  
```bash
sudo apt update
sudo apt install -y git make gcc g++ libsctp-dev lksctp-tools
```  

### **3. Clone & Build UERANSIM**  
```bash
git clone https://github.com/aligungr/UERANSIM
cd UERANSIM
make
```  

---

## **Step 4: Configure UERANSIM (VM) for Open5GS (Host)**  
### **1. Edit `gnb.yaml`**  
```yaml
plmn_list:
  - mcc: "001"   # Must match Open5GS AMF
    mnc: "01"

amf_configs:
  - address: 192.168.122.1  # Host’s NAT IP (gateway)
    port: 38412             # NGAP port
```  

### **2. Edit `ue.yaml`**  
```yaml
supi: "imsi-001010000000001"
plmn: "00101"               # MCC=001, MNC=01
op: "8e27b6af0e692e750f32667a3b14605d"  # Default Open5GS key
```  

---

## **Step 5: Configure Open5GS (Host) for VM Access**  
### **1. Edit `amf.yaml`**  
```yaml
amf:
  ngap:
    server:
      - address: 0.0.0.0  # Listen on all interfaces
        port: 38412
```  
- **Restart AMF**:  
  ```bash
  sudo systemctl restart open5gs-amfd
  ```  

### **2. Allow SCTP Traffic**  
```bash
sudo ufw allow 38412/sctp
```  

---

## **Step 6: Start UERANSIM and Test**  
### **1. In VM**  
```bash
./nr-gnb -c config/gnb.yaml  # Start gNB
./nr-ue -c config/ue.yaml    # Start UE
```  

### **2. Verify Connectivity**  
- **Check AMF Logs (Host)**:  
  ```bash
  journalctl -u open5gs-amfd -f
  ```  
  - Expected:  
    ```
    NGAP connection from 192.168.122.234
    ```  
- **Test UE (VM)**:  
  ```bash
  ./nr-ue -c config/ue.yaml --ping 8.8.8.8
  ```  

---

## **Troubleshooting Your Issues**  
### **1. "SCTP Network Unreachable"**  
**Cause**:  
- AMF not bound to `0.0.0.0`.  
- Firewall blocking SCTP.  

**Fix**:  
```bash
sudo ufw allow 38412/sctp  # Allow SCTP
sudo systemctl restart open5gs-amfd
```  

### **2. "Unhandled SCTP Notifications"**  
**Cause**: Benign SCTP protocol messages.  
**Fix**: Ignore if UE attaches successfully.  

### **3. Ping Works but SCTP Fails**  
**Debug**:  
```bash
# On VM:
nc -zv 192.168.122.1 38412 -u  # Test SCTP
```  
- If **fails**: Check AMF logs and firewall.  

