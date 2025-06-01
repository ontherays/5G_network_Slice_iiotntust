## ✅ **System Overview**

* **Core Network:** Open5GS (built from source)
* **UE & RAN Simulator:** UERANSIM (v3.2.7)
* **MCC/MNC Used:** `001/01`
* **Installation Mode:** Built from source, using custom install path

  * **Binary path:** `/home/ravi/open5gs/install/bin/`
  * **Config path:** `/home/ravi/open5gs/install/etc/open5gs/`

---

## 📦 **1. Built and Installed Open5GS**

You followed the official guide to build Open5GS from source:
[https://open5gs.org/open5gs/docs/guide/02-building-open5gs-from-sources/](https://open5gs.org/open5gs/docs/guide/02-building-open5gs-from-sources/)

Commands (example):

```bash
git clone https://github.com/open5gs/open5gs
cd open5gs
meson build --prefix=/home/ravi/open5gs/install
ninja -C build
ninja -C build install
```

---

## 🌐 **2. Configured WebUI (Optional UI)**

Built and launched Open5GS WebUI (for managing subscribers, etc.) if installed.

---

## ⚙️ **3. Created systemd Service Files**

Created systemd unit files for all **Open5GS NFs** manually.

### Services Created:

You created systemd service files for all the following NFs under:
`/etc/systemd/system/open5gs-*.service`

| Service               | Daemon               |
| --------------------- | -------------------- |
| open5gs-nrfd.service  | open5gs-nrfd         |
| open5gs-bsfd.service  | open5gs-bsfd         |
| open5gs-udrd.service  | open5gs-udrd         |
| open5gs-udmd.service  | open5gs-udmd         |
| open5gs-udm.service   | (incorrect, deleted) |
| open5gs-ausfd.service | open5gs-ausfd        |
| open5gs-nssfd.service | open5gs-nssfd        |
| open5gs-pcfd.service  | open5gs-pcfd         |
| open5gs-amfd.service  | open5gs-amfd         |
| open5gs-smfd.service  | open5gs-smfd         |
| open5gs-upfd.service  | open5gs-upfd         |
| open5gs-scpd.service  | open5gs-scpd         |
| open5gs-seppd.service | open5gs-seppd        |
| open5gs-sgwcd.service | open5gs-sgwcd        |
| open5gs-sgwud.service | open5gs-sgwud        |
| open5gs-mmed.service  | open5gs-mmed         |
| open5gs-pcrfd.service | open5gs-pcrfd        |
| open5gs-hssd.service  | open5gs-hssd         |

🛠️ Deleted duplicate or incorrect:

```bash
sudo rm /etc/systemd/system/open5gs-udm.service
```

---

## 🔄 **4. Enabled All Services for Boot**

Commands executed:

```bash
sudo systemctl enable open5gs-nrfd.service
sudo systemctl enable open5gs-bsfd.service
sudo systemctl enable open5gs-udrd.service
sudo systemctl enable open5gs-udmd.service
sudo systemctl enable open5gs-ausfd.service
sudo systemctl enable open5gs-nssfd.service
sudo systemctl enable open5gs-pcfd.service
sudo systemctl enable open5gs-amfd.service
sudo systemctl enable open5gs-smfd.service
sudo systemctl enable open5gs-upfd.service
sudo systemctl enable open5gs-scpd.service
sudo systemctl enable open5gs-seppd.service
sudo systemctl enable open5gs-sgwcd.service
sudo systemctl enable open5gs-sgwud.service
sudo systemctl enable open5gs-mmed.service
sudo systemctl enable open5gs-pcrfd.service
sudo systemctl enable open5gs-hssd.service
```

---

## 🚀 **5. Custom Start Script**

Created custom `start-open5gs.sh` script with ordered startup:

```bash
#!/bin/bash
# start-open5gs.sh

services=(
  open5gs-nrfd
  open5gs-bsfd
  open5gs-udrd
  open5gs-udmd
  open5gs-ausfd
  open5gs-nssfd
  open5gs-pcfd
  open5gs-amfd
  open5gs-smfd
  open5gs-upfd
)

echo "▶️ Starting all Open5GS 5GC services in correct order..."
for svc in "${services[@]}"; do
  echo "Starting $svc.service"
  sudo systemctl start "$svc.service"
  sleep 1
done

echo "📋 Checking status of all Open5GS 5GC services..."
for svc in "${services[@]}"; do
  echo "----------------------------------------"
  echo "🔍 Status: $svc.service"
  systemctl status "$svc.service" --no-pager --lines=5
done
```

---

## ⛔ **6. Fixed UDM Issue**

* `open5gs-udm.service` was a duplicate and non-functional.
* Kept `open5gs-udmd.service` and removed the invalid `.udm.service`.

---

## 📡 **7. Ran UERANSIM (UE)**

Configured and ran UE:

```bash
cd ~/UERANSIM
sudo ./build/nr-ue -c config/open5gs-ue.yaml
```

Log confirmed:

* PLMN selected ✅
* Registration successful ✅
* Authentication completed ✅
* PDU session accepted ✅
* Interface `uesimtun0` assigned IP: `10.45.0.3` ✅

---

## 🔚 **You now have:**

* ✅ A complete 5GC setup (Open5GS) running via systemd.
* ✅ Ordered startup for NFs via custom script.
* ✅ Valid UE registration and data session using UERANSIM.

---

## ✅ **1. Setup Overview**

### ✔️ Components

* **Open5GS 5GC**: Built from source and running on your lab PC.
* **UERANSIM**: Used to simulate UE and connect to the 5GC.
* **Amazon EC2**: Remote server used as `iperf3` server.
* **Interface Setup**:

  * `ogstun` – Core network interface for 5GC.
  * `uesimtun0` – Created by UERANSIM, IP: `10.45.0.2`

---

## ✅ **2. Local Network Tests**

### ✔️ Ping from host

```bash
ping 10.45.0.2     # ✅ success
ping 8.8.8.8       # ✅ success from host
ping -I uesimtun0 8.8.8.8  # ❌ failed
```

### ✔️ Reason: No NAT for `uesimtun0`

---

## ✅ **3. NAT Configuration for UE Traffic**

### ✔️ Enable IP forwarding

```bash
sudo sysctl -w net.ipv4.ip_forward=1
echo "net.ipv4.ip_forward = 1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

### ✔️ Set NAT on correct interface (`wlp58s0`)

```bash
sudo iptables -t nat -A POSTROUTING -s 10.45.0.0/16 -o wlp58s0 -j MASQUERADE
```

---

## ✅ **4. EC2 Instance Setup for `iperf3`**

### ✔️ EC2 Configuration

* **AMI**: Ubuntu 22.04
* **Security Group Rules**:

  ```text
  Inbound Rule:
  Type: Custom TCP
  Port: 5201
  Source: <Your Public IP>/32
  ```

### ✔️ Start iperf3 server on EC2

```bash
iperf3 -s
```

---

## ✅ **5. iperf3 Tests**

### ❌ Initial attempt (failed):

```bash
iperf3 -c <EC2_PUBLIC_IP> -B 10.45.0.2 -t 10
# Error: unable to send control message: Bad file descriptor
```

### ✔️ Wireshark Debug:

* TCP SYN from `10.45.0.2` seen.
* No SYN-ACK → EC2 didn't respond.

### ✔️ Root Cause:

* EC2 never sees a public IP.
* Without NAT, UE IP `10.45.0.2` is **non-routable**.

---

## ✅ **6. Fix: NAT and Route Verification**

### ✔️ Verify routing

```bash
ip route get 8.8.8.8
# Confirmed outgoing via wlp58s0
```

### ✔️ iptables verification

```bash
sudo iptables -t nat -L -n -v
```

---

## ✅ **7. Final `iperf3` Test Plan**

### After NAT is set:

```bash
ping -I uesimtun0 8.8.8.8         # ✅ should succeed
iperf3 -c <EC2_PUBLIC_IP> -B 10.45.0.2 -t 10  # ✅ should now work
```

---

## ✅ **8. iperf3 for Latency + Jitter (Optional)**

```bash
# UDP test for jitter and packet loss
iperf3 -c <EC2_PUBLIC_IP> -B 10.45.0.2 -u -b 1M -t 10
```


## Issue and check logs

Here's a **summary of the last 5 responses**, including **commands and their purpose**:

---

### ✅ **1. Check Why SMF Failed**

**Command:**

```bash
/home/ravi/open5gs/install/bin/open5gs-smfd -c /home/ravi/open5gs/install/etc/open5gs/smf.yaml
```

**Purpose:**
Run the SMF binary manually to view the real-time error message that caused the systemd service to fail. This helps debug YAML or config issues.

---

### ✅ **2. Validate YAML Format**

**Command (optional but recommended):**

```bash
sudo apt install yamllint
yamllint /home/ravi/open5gs/install/etc/open5gs/smf.yaml
```

**Purpose:**
Check for YAML formatting issues like incorrect indentation or invalid structure, which can crash Open5GS services.

---

### ✅ **3. Correct `smf.yaml` Network Slice Syntax**

**Key Configuration:**

```yaml
session:
  - subnet: 10.45.0.0/16
    gateway: 10.45.0.1
    dnn: internet
    s_nssai:
      - sst: 1
        sd: 010203
      - sst: 1
        sd: 112233
```

**Purpose:**
Ensure SMF is configured to recognize two different network slices by SST/SD for slice-aware IP allocation and traffic control.

---

### ✅ **4. Reload and Restart the SMF Service**

**Commands:**

```bash
sudo systemctl daemon-reexec
sudo systemctl daemon-reload
sudo systemctl restart open5gs-smfd.service
```

**Purpose:**
Apply recent changes to the systemd unit and restart the SMF cleanly after fixing the config.

---

### ✅ **5. General Troubleshooting Tip**

**Website:**
[https://codebeautify.org/yaml-validator](https://codebeautify.org/yaml-validator)

**Purpose:**
Provides an online way to check YAML files quickly for syntax errors that might break service startup.


---

### Check network fuctions logs

sudo systemctl restart open5gs-amfd <replace netwok function as required>

sudo systemctl status open5gs-amfd


