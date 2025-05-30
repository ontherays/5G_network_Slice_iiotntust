Here's a comprehensive, step-by-step summary of the entire configuration and setup process you performed today to get Open5GS working with UERANSIM for a successful PDU session establishment:

---

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

Let me know if you'd like a backup export of your YAML config files or help scripting auto-testing later.
