# AWS EC2 Configuration for V2X Emulation

## 1. Instance Specifications
yaml

Type: c5.2xlarge (8 vCPUs, 16GB RAM)
AMI: Ubuntu 22.04 LTS (ami-0c55b159cbfafe1f0)
Storage: 50GB GP3 EBS
Security Group: 
  - Inbound: TCP 22, 80, 7777, 38412, 2152, 9080
  - Outbound: All traffic
Elastic IP: 54.210.178.33 (replace with your actual IP)

## 2. Network Configuration
bash

# Configure secondary IPs (for multiple UEs)
sudo ip addr add 10.45.0.100/24 dev eth0  # UPF IP
sudo ip addr add 10.45.0.1/24 dev eth0     # UE1
sudo ip addr add 10.45.0.2/24 dev eth0     # UE2

## 3. Core Components Installation
bash

### Open5GS Core
sudo add-apt-repository ppa:open5gs/latest
sudo apt install open5gs
sudo cp /home/ubuntu/open5gs/install/etc/open5gs/amf.yaml /etc/open5gs/

### UERANSIM
sudo apt install make gcc g++ libsctp-dev lksctp-tools
git clone https://github.com/aligungr/UERANSIM
cd UERANSIM && make

## 4. Configuration Files

amf.yaml (Critical Excerpt)
yaml

amf:
  sbi:
    server:
      - address: 10.45.0.100  # UPF IP
  ngap:
    server:
      - address: 10.45.0.100
  guami:
    - plmn_id: {mcc: 001, mnc: 01}
  plmn_support:
    - plmn_id: {mcc: 001, mnc: 01}
      s_nssai:
        - sst: 1
          sd: "010203"

gnb.yaml
yaml

mcc: '001'
mnc: '01'
linkIp: 10.45.0.100  # UPF IP
ngapIp: 10.45.0.100
amfConfigs:
  - address: 10.45.0.100
    port: 38412
    s_nssai:
      - sst: 1
        sd: "010203"

## 5. AWS IoT Core Setup
bash

# Create IoT Policy
aws iot create-policy --policy-name V2X-Policy \
  --policy-document '{
    "Version":"2012-10-17",
    "Statement":[{
      "Effect":"Allow",
      "Action":["iot:*"],
      "Resource":"*"
    }]
  }'

## 6. Test Execution
bash

# Start components in order:
1. Open5GS: sudo systemctl start open5gs-amfd open5gs-upfd
2. gNB: ./nr-gnb -c config/open5gs-gnb.yaml
3. UE1: ./nr-ue -c config/ue1.yaml
4. UE2: ./nr-ue -c config/ue2.yaml

### Verify connections:
ping 10.45.0.1  # UE1
ping 10.45.0.2  # UE2

## 7. Monitoring Setup
bash

### Install monitoring tools
sudo apt install prometheus-node-exporter
sudo systemctl enable prometheus-node-exporter

# Grafana Dashboard (Port 3000)
wget https://raw.githubusercontent.com/open5gs/open5gs/main/misc/grafana/5gc.json

Key Learnings from Your Tests:

    The slice configuration (S-NSSAI) must match exactly between gNB and AMF

    Network name parameters are case-sensitive

    EC2 instance needs enhanced networking enabled for GTP-U performance

    Always start AMF before gNB to avoid SCTP connection issues

Troubleshooting Checklist:
bash

# Check AMF-gNB connection
sudo tcpdump -i any port 38412 -n

# Verify NRF registration
curl http://10.45.0.100:7777/nnrf-nfm/v1/nf-instances | jq .

# Check UE registration status
open5gs-dbctl show
