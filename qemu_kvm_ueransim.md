### **Summary: QEMU/KVM Installation & UERANSIM Configuration on Ubuntu 22.04**  
This guide provides a **detailed, step-by-step** process to:  
1. Set up a **QEMU/KVM virtual machine (VM)** on Ubuntu 22.04.  
2. Install and configure **UERANSIM (gNB + UE)** inside the VM.  
3. Establish **end-to-end connectivity** with Open5GS (host machine).  

---

## **Step 1: Install QEMU/KVM on Host**  
### **1. Check Virtualization Support**  
```bash
egrep -c '(vmx|svm)' /proc/cpuinfo  # Output ≥1 means supported
```  
- Enable **VT-x/AMD-V** in BIOS if needed.  

### **2. Install Required Packages**  
```bash
sudo apt update
sudo apt install -y qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils virt-manager
```  

### **3. Add User to libvirt & KVM Groups**  
```bash
sudo usermod -aG libvirt $(whoami)
sudo usermod -aG kvm $(whoami)
```  
- **Log out and back in** to apply changes.  

### **4. Verify Installation**  
```bash
virsh list --all  # Should show no errors
virt-manager      # Launch GUI (optional)
```  

---

## **Step 2: Create a VM for UERANSIM**  
### **1. Download Ubuntu ISO**  
- Get Ubuntu 22.04 Server/Desktop ISO:  
  ```bash
  wget https://releases.ubuntu.com/22.04/ubuntu-22.04.3-live-server-amd64.iso
  ```  

### **2. Create VM via `virt-manager`**  
- Open `virt-manager` → **New VM** → Select ISO.  
- Allocate resources:  
  - **CPU**: 2+ cores  
  - **RAM**: 4GB+  
  - **Storage**: 20GB+ (QCOW2 format)  
- Select **NAT** or **Bridged Networking** (see below).  

### **3. Configure Networking**  
#### **Option A: NAT (Default)**  
- VM gets IP from `192.168.122.0/24` subnet.  
- Host acts as gateway (`192.168.122.1`).  

#### **Option B: Bridged Networking**  
- VM gets IP from the **same subnet as host** (e.g., `192.168.1.x`).  
- Edit `/etc/netplan/01-netcfg.yaml` on host:  
  ```yaml
  network:
    version: 2
    renderer: networkd
    ethernets:
      enp3s0:           # Replace with your interface
        dhcp4: no
    bridges:
      br0:
        interfaces: [enp3s0]
        dhcp4: yes
  ```  
  Apply:  
  ```bash
  sudo netplan apply
  ```  

---

## **Step 3: Install UERANSIM in VM**  
### **1. SSH into VM**  
```bash
virsh console <VM_NAME>  # Or use virt-manager GUI
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

### **4. Configure gNB and UE**  
#### **Edit `gnb.yaml`**  
```yaml
plmn_list:
  - mcc: "001"   # Match Open5GS AMF config
    mnc: "01"

amf_configs:
  - address: 192.168.122.1  # Host’s NAT IP (if using NAT)
    port: 38412             # Default NGAP port
```  

#### **Edit `ue.yaml`**  
```yaml
supi: "imsi-001010000000001"
plmn: "00101"
op: "8e27b6af0e692e750f32667a3b14605d"  # Default Open5GS key
```  

---

## **Step 4: Connect UERANSIM (VM) to Open5GS (Host)**  
### **1. Host Configuration (Open5GS AMF)**  
Ensure `amf.yaml` listens on **all interfaces**:  
```yaml
amf:
  ngap:
    server:
      - address: 0.0.0.0  # Allow VM connections
        port: 38412
```  
Restart AMF:  
```bash
sudo systemctl restart open5gs-amfd
```  

### **2. Allow SCTP Traffic on Host**  
```bash
sudo ufw allow 38412/sctp
```  

### **3. Start UERANSIM in VM**  
```bash
./nr-gnb -c config/gnb.yaml  # Start gNB
./nr-ue -c config/ue.yaml    # Start UE
```  

---

## **Step 5: Verify Connectivity**  
### **1. Check AMF Logs (Host)**  
```bash
journalctl -u open5gs-amfd -f
```  
- Look for:  
  ```
  NGAP connection from [VM_IP]
  ```  

### **2. Test UE Registration**  
```bash
./nr-ue -c config/ue.yaml --ping 8.8.8.8  # From VM
```  

---

## **Troubleshooting**  
| Issue                          | Solution                                  |
|--------------------------------|------------------------------------------|
| **SCTP connection failed**     | Check firewall (`sudo ufw allow 38412/sctp`) |
| **AMF unreachable**           | Ensure AMF listens on `0.0.0.0`          |
| **UE fails to attach**        | Verify PLMN/SUPI matches Open5GS config  |
| **No IP in VM**               | Restart libvirt NAT: `sudo virsh net-restart default` |

---

### **Final Notes**  
- **NAT**: Simpler, but VM is isolated from LAN.  
- **Bridged**: VM behaves like a physical device on the network.  
- For **production**, use **static IPs** (via `virsh net-edit default`).  

This setup ensures **full 5G core (Open5GS) + RAN (UERANSIM)** interoperability. Let me know if you need further refinements! 🚀
