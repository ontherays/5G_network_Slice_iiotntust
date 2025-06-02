### System Information

| Component         | Recommended Specification                  |
|-------------------|--------------------------------------------|
| OS                | Ubuntu 20.04 LTS (minimal)                 |
| Kernel            | 5.4.0 (default in Ubuntu 20.04)            |
| CPU               | 4+ cores                                  |
| RAM               | 8+ GB                                     |
| Network           | Internet access (for downloads)           |
| Logs              | Wireshark          |


### Architecture
![System architecture](https://github.com/user-attachments/assets/7f664c23-9950-4f90-a535-5c3865dac649)

### Sytem Architecture

![image](https://github.com/user-attachments/assets/8269af39-9357-47a9-a5fa-a815253f1b51)


### Topology
![topology](https://github.com/user-attachments/assets/ed1aea41-36b8-4010-bebe-d5d25034adea)


### **Key Components**:
1. **UE (📱)**:
   - UERANSIM UE with TUN interface `uesimtun0`
   - Assigned IP: `10.45.0.3`

2. **gNB (🏭)**:
   - UERANSIM gNodeB
   - N2 (NGAP): `10.10.0.5:38412`
   - N3 (GTP-U): `10.10.0.5:2152`

3. **Core Network**:
   - **AMF (🖥️)**: `127.0.0.5:7777` (SBI), `10.10.0.5:38412` (NGAP)
   - **SMF (🧠)**: `127.0.0.4:7777` (SBI)
   - **UPF (🌐)**: `127.0.0.7:8805` (N4), `10.10.0.5:2152` (N3)

### **Protocol Interfaces**:
- **N1/N2**: UE↔gNB (NAS/RRC)
- **NGAP**: gNB↔AMF (SCTP)
- **GTP-U**: gNB↔UPF (User Plane)
- **N4**: SMF↔UPF (PFCP)
- **N11**: AMF↔SMF (HTTP2)

## Building Open5GS from Sources

### Getting MongoDB

- Import the public key used by the package management system.

   $ sudo apt update
   $ sudo apt install gnupg
   $ curl -fsSL https://pgp.mongodb.com/server-8.0.asc | sudo gpg -o /usr/share/keyrings/mongodb-server-8.0.gpg --dearmor

- Create the list file /etc/apt/sources.list.d/mongodb-org-8.0.list for your version of Ubuntu. On ubuntu 22.04 

   $ echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-8.0.gpg] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/8.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-8.0.list

- Install the MongoDB packages.

   $ sudo apt update
   $ sudo apt install -y mongodb-org
   $ sudo systemctl start mongod (if '/usr/bin/mongod' is not running)
   $ sudo systemctl enable mongod (ensure to automatically start it on system boot)

### Setting up TUN device (not persistent after rebooting)

- Create the TUN device with the interface name ogstun.

   $ sudo ip tuntap add name ogstun mode tun
   $ sudo ip addr add 10.45.0.1/16 dev ogstun
   $ sudo ip addr add 2001:db8:cafe::1/48 dev ogstun
   $ sudo ip link set ogstun up

### Building Open5GS

- **Install the common dependencies for building the source code.**

   $ sudo apt install python3-pip python3-setuptools python3-wheel ninja-build build-essential flex bison git cmake libsctp-dev libgnutls28-dev libgcrypt-dev libssl-dev libmongoc-dev libbson-dev libyaml-dev libnghttp2-dev libmicrohttpd-dev libcurl4-gnutls-dev libnghttp2-dev libtins-dev libtalloc-dev meson

Install libidn-dev or libidn11-dev depending on your system

   $ if apt-cache show libidn-dev > /dev/null 2>&1; then
      sudo apt-get install -y --no-install-recommends libidn-dev
   else
      sudo apt-get install -y --no-install-recommends libidn11-dev
   fi

- **Git clone.**

   $ git clone https://github.com/open5gs/open5gs

- To compile with meson:

   $ cd open5gs
   $ meson build --prefix=`pwd`/install
   $ ninja -C build

- Check whether the compilation is correct.

   $ ./build/tests/attach/attach ## for EPC Only
   $ ./build/tests/registration/registration ## for 5G Core Only

- Run all test programs as below.

   $ cd build
   $ meson test -v

- You need to perform the installation process.

   $ cd build
   $ ninja install
   $ cd ../

- Configure Open5GS

   5G Core

   Modify install/etc/open5gs/nrf.yaml to set the NGAP IP address, PLMN ID, TAC and NSSAI.

### Configuration Table

### Open5GS & UERANSIM Configuration Reference

| Network Element | Configuration File (YAML) | Key Parameters | Values |
|-----------------|--------------------------|----------------|----------------|
| **AMF** (Open5GS) | `open5gs/install/etc/open5gs/amf.yaml` | `ngap:`<br>`  addr: <NGAP_IP>`<br>`plmn_id:`<br>`  mcc: <MCC>`<br>`  mnc: <MNC>`<br>`tac: <TAC>`<br>`slice:`<br>`  sst: 1`<br>`  sd: <SD>` | `addr: 127.0.0.5`<br>`mcc: 001`<br>`mnc: 01`<br>`tac: 1`<br>`sst: 1`<br>`sd: 0xFFFFFF` |
| **SMF** (Open5GS) | `open5gs/install/etc/open5gs/smf.yaml` | `sbi:`<br>`  addr: <SMF_IP>`<br>`upf:`<br>`  - addr: <UPF_IP>`<br>`dnn:`<br>`  - name: <DNN>` | `addr: 192.168.1.101`<br>`addr: 192.168.1.102`<br>`name: internet` |
| **UPF** (Open5GS) | `open5gs/install/etc/open5gs/upf.yaml` | `gtpu:`<br>`  addr: <GTPU_IP>`<br>`dnn:`<br>`  - name: <DNN>` | `addr: 192.168.1.102`<br>`name: internet` |
| **gNB** (UERANSIM) | `UERANSIM/config/gNB.yaml` | `linkIp: <NGAP_IP>`<br>`ngapIp: <NGAP_IP>`<br>`gtpIp: <GTPU_IP>`<br>`plmn:`<br>`  mcc: <MCC>`<br>`  mnc: <MNC>`<br>`tac: <TAC>`<br>`slices:`<br>`  - sst: <SST>`<br>`    sd: <SD>` | `linkIp: 192.168.1.200`<br>`ngapIp: 192.168.1.200`<br>`gtpIp: 192.168.1.200`<br>`mcc: 001`<br>`mnc: 01`<br>`tac: 7`<br>`sst: 1`<br>`sd: 0xFFFFFF` |
| **UE** (UERANSIM) | `UERANSIM/config/UE.yaml` | `supi: <SUPI>`<br>`plmn:`<br>`  mcc: <MCC>`<br>`  mnc: <MNC>`<br>`sessions:`<br>`  - type: <PDN_TYPE>`<br>`    apn: <DNN>`<br>`    slice:`<br>`      sst: <SST>`<br>`      sd: <SD>` | `supi: imsi-001010000000001`<br>`mcc: 001`<br>`mnc: 01`<br>`type: IPv4`<br>`apn: internet`<br>`sst: 1`<br>`sd: 0xFFFFFF` |

### Key Parameters Explained:
- **NGAP IP**: IP address for NGAP interface (AMF <-> gNB communication).
- **PLMN ID**: Public Land Mobile Network ID (`MCC` + `MNC`).
- **TAC**: Tracking Area Code (e.g., `7`).
- **NSSAI**: Network Slice Selection Assistance Information (`SST` + `SD`).
  - **SST**: Slice/Service Type (e.g., `1` for eMBB).
  - **SD**: Slice Differentiator (optional, e.g., `0xFFFFFF`).
- **DNN**: Data Network Name (e.g., `internet`).

### Example Network Setup:
- **AMF NGAP IP**: `192.168.1.100`
- **gNB NGAP IP**: `192.168.1.200`
- **PLMN ID**: `001-01`
- **TAC**: `1`
- **NSSAI**: `SST=1`, `SD=0xFFFFFF`


### Running Open5GS

 $ sudo nano start_check_open5gs.sh

```
      #!/bin/bash
# start-open5gs.sh - Starts Open5GS 5GC network functions in correct order (for 5G SA)

# Ordered list of Open5GS 5GC services, respecting typical NF dependencies and startup order
services=(
  open5gs-nrfd    # NRF must start first
  open5gs-bsfd    # BSF
  open5gs-udrd    # UDR
  open5gs-udmd    # UDM
  open5gs-ausfd   # AUSF
  open5gs-nssfd   # NSSF
  open5gs-pcfd    # PCF
#  open5gs-pcrfd   # PCRF (Policy Control Rule Function)
  open5gs-seppd   # SEPP (Security Edge Protection Proxy)
  open5gs-amfd    # AMF (after AUSF, NSSF, UDM, etc.)
  open5gs-mmed    # MME-D (for EPS, optional in 5G SA)
  open5gs-smfd    # SMF (after AMF)
  open5gs-scpd    # SCPD (Service Communication Proxy Daemon)
  open5gs-sgwcd   # SGW-C (Serving Gateway Control Plane)
  open5gs-sgwud   # SGW-U (Serving Gateway User Plane)
  open5gs-upfd    # UPF (after SMF, SGW-U)
)

echo "▶️ Starting all Open5GS 5GC services in correct order..."
for svc in "${services[@]}"; do
  echo "Starting $svc.service"
  sudo systemctl start "$svc.service"
  sleep 1
done

echo ""
echo "📋 Checking status of all Open5GS 5GC services..."
for svc in "${services[@]}"; do
  echo "----------------------------------------"
  echo "🔍 Status: $svc.service"
  systemctl status "$svc.service" --no-pager --lines=5
done

```

### Stop of Open5GS

   $ sudo nano stop_open5gs.sh

 ```
      #!/bin/bash
# stop-open5gs.sh - Stops all Open5GS 5GC services in reverse order

services=(
  open5gs-upfd
  open5gs-sgwud
  open5gs-sgwcd
  open5gs-scpd
  open5gs-smfd
  open5gs-mmed
  open5gs-amfd
  open5gs-seppd
  open5gs-pcrfd
  open5gs-pcfd
  open5gs-nssfd
  open5gs-ausfd
  open5gs-udmd
  open5gs-udrd
  open5gs-bsfd
  open5gs-nrfd
)

echo "⏹️ Stopping all Open5GS 5GC services in reverse order..."
for svc in "${services[@]}"; do
  echo "Stopping $svc.service"
  sudo systemctl stop "$svc.service"
done

echo "✅ All services stopped."

 ```

$ chmod +x stop_open5gs.sh #permision
$ ./stop_open5gs.sh # run


### Building the WebUI of Open5GS

   Node.js is required to build WebUI of Open5GS

      # Download and import the Nodesource GPG key
      $ sudo apt update
      $ sudo apt install -y ca-certificates curl gnupg
      $ sudo mkdir -p /etc/apt/keyrings
      $ curl -fsSL https://deb.nodesource.com/gpgkey/nodesource-repo.gpg.key | sudo gpg --dearmor -o /etc/apt/keyrings/nodesource.gpg

   ### Create deb repository
      $ NODE_MAJOR=20
      $ echo "deb [signed-by=/etc/apt/keyrings/nodesource.gpg] https://deb.nodesource.com/node_$NODE_MAJOR.x nodistro main" | sudo tee /etc/apt/sources.list.d/nodesource.list

   ### Run Update and Install
      $ sudo apt update
      $ sudo apt install nodejs -y

      Install the dependencies to run WebUI

      $ cd webui
      $ npm ci

   **The WebUI runs as an npm script.**

      $ npm run dev

   Server listening can be changed by setting the environment variable HOSTNAME or PORT as below.

      $ HOSTNAME=192.168.0.11 npm run dev
      $ PORT=7777 npm run dev



### Register Subscriber Information

   Connect to http://127.0.0.1:9999 and login with admin account.

    Username : admin
    Password : 1423

   Edit 'imsi-001010000000001'

   ![Screenshot 2025-05-04 134254](https://github.com/user-attachments/assets/5b52a27a-fd7f-4c24-b6f4-6ad84eaecc2d)

   Setting up subscribers

   <img width="924" alt="image" src="https://github.com/user-attachments/assets/468fe548-7f62-46f3-9fc8-11ebd5c9090c" />


