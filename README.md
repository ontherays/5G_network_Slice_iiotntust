## 5G_network_Slice_iiotntust
This project focuses on implementing and validating a 5G-based V2X (Vehicle-to-Everything) and V2V (Vehicle-to-Vehicle) communication framework, where vehicles are emulated using User Equipments (UEs) through UERANSIM. The core network is implemented using Open5GS, and UEs are tested in various scenarios, including on-host, QEMU virtualized environments, and integration with external IoT devices hosted on AWS. The testing involves validating full end-to-end (E2E) communication, slice-based QoS differentiation, and overall network responsiveness and efficiency.

# System Framework

![image](https://github.com/user-attachments/assets/23f853d6-235e-4e80-8a8c-c952af0f660a)

## System Architecture

![Screenshot from 2025-06-03 17-37-55](https://github.com/user-attachments/assets/438eba2a-3910-48d8-bbeb-9aea6f651881)


### Key Updates & Annotations

  IP Addressing

    UE (Vehicle1): 10.45.0.1/24
    
    UE (Vehicle2): 10.45.0.2/24 (Fixed typo from your input: both UEs can’t have same IP)
    
    UPF (Open5GS): 10.45.0.100/24 (Gateway for UEs)

  Icons for Clarity

    - AWS IoT Core
  
    - V2X Application Server
  
    - Database (DynamoDB/Timestream)
  
    - Analytics (QuickSight/Grafana)
  
    - AMF (Control Plane)
  
    - SMF (Session Management)
  
    - UPF (User Plane)
  
    -  gNodeB (RAN)
  
    🚗/🚙: UEs (Vehicles)
  
    🔄: V2V Sidelink Communication

Interfaces Labeled

    N1/N2: NGAP (UE ↔ AMF, gNodeB ↔ AMF)

    N3: GTP-U (gNodeB ↔ UPF)

    PC5: Sidelink (Direct V2V)

---

### **Technical Project Description: Emulation and Testing of V2X/V2V Communications Using Open5GS and UERANSIM**

#### **Project Overview**

This project focuses on implementing and validating a 5G-based V2X (Vehicle-to-Everything) and V2V (Vehicle-to-Vehicle) communication framework, where vehicles are emulated using User Equipments (UEs) through UERANSIM. The core network is implemented using Open5GS, and UEs are tested in various scenarios, including on-host, QEMU virtualized environments, and integration with external IoT devices hosted on AWS. The testing involves validating full end-to-end (E2E) communication, slice-based QoS differentiation, and overall network responsiveness and efficiency.

---

### **Goals**

1. Emulate vehicles as 5G UEs using UERANSIM and validate their interaction with the 5G core network (Open5GS).
2. Establish and test UE-to-UE communication for V2V scenarios.
3. Connect UEs with IoT devices hosted on AWS to simulate V2X (e.g., V2I and V2C) interactions.
4. Deploy UERANSIM UEs in virtualized environments (QEMU) and verify their registration and data paths.
5. Implement and test network slicing for differentiated QoS—one slice for low-latency and another for high-throughput communication.
6. Measure performance gains in terms of throughput and latency to validate the network's effectiveness in real-world V2X use cases.

---

### **System Architecture and Configuration**

**Core Components:**

* **Open5GS** deployed from source on a host machine with dedicated paths:

  * Binary: `/home/ravi/open5gs/install/bin/`
  * Config: `/home/ravi/open5gs/install/etc/open5gs/`
* **UERANSIM** compiled on both the host and within a QEMU virtual machine to simulate UEs as vehicles.
* **gNB** configuration hosted separately on both the host and VM with proper NGAP and GTP IP bindings.
* **AWS IoT Devices** connected over public IP from UEs for V2X interaction testing.

**Network Setup:**

* MCC: 001, MNC: 01 for the test PLMN.
* IP subnet: 10.45.0.0/16 used for PDU sessions with UE gateways at 10.45.0.1.
* `uesimtun0` created on UEs after successful PDU establishment.
* `ogstun` used on the UPF for routing PDU traffic to/from the internet and peer UEs.
* Firewall (`ufw`) disabled to ensure open communication paths.
* UPF `gtpu` server address set to `0.0.0.0` to enable external gNB (on VM) communication.

**gNB (UERANSIM) Configuration:**

* `linkIp`, `ngapIp`, and `gtpIp` bound to local VM IP: `192.168.122.234`
* `amfConfigs` pointing to host machine IP: `192.168.122.1`
* Multiple slices defined using `sst: 1` and `sst: 2`

**Testing Environment:**

* UE1 on host machine
* UE2 on QEMU VM with bridge networking for seamless IP communication
* AMF, SMF, UPF, AUSF, UDM, PCF all managed as systemd services with monitored logs

---

### **Testing Procedures**

**1. Registration and PDU Session Tests:**

Each UE, whether hosted or virtualized, is tested to:

* Register successfully with the Open5GS AMF
* Receive a valid IP (10.45.0.X) on `uesimtun0`
* Establish PDU session and verify tunnel interface is up
* Use `ping` and `iperf3` to test data path to core and internet

  ![messageflow](https://github.com/user-attachments/assets/c08535a2-5abf-4260-a359-62c954b25d53)

**2. UE to UE Communication (V2V):**

* Both UEs connected to the same 5G core
* IPs assigned within same subnet
* Tested connectivity over `uesimtun0` interfaces
* Static routes added where required


**3. UE to IoT Devices (V2X over Cloud):**

* UEs ping AWS IoT endpoints
* Use `iperf3` from UE to EC2 instance to simulate telemetry or alert system in V2X
* Measured latency and throughput

**4. Slice Differentiation Testing:**

Two slices configured on the network:

* **Slice 1 (sst: 1)**: Prioritized for **throughput**; used by UE1
* **Slice 2 (sst: 2)**: Prioritized for **latency**; used by UE2

Testing included:

* Simultaneous `iperf3` traffic generation on both slices
* Measuring response times and throughput statistics
* Observing how slice isolation helps maintain low latency even under load

---

### **Results and Achievements**

| Test Scenario     | UE         | Slice       | Expected Goal     | Achieved Result |
| ----------------- | ---------- | ----------- | ----------------- | --------------- |
| UE1 Registration  | UE1 (Host) | sst: 1      | Successful Attach | Passed          |
| UE2 Registration  | UE2 (VM)   | sst: 2      | Successful Attach | Passed          |
| UE to Open5GS     | UE1/UE2    | Both        | Full PDU Setup    | Passed          |
| UE1 to UE2 (Ping) | UE1 ↔ UE2  | Cross-slice | <20ms RTT         | \~15ms          |
| UE1 to AWS        | UE1        | sst: 1      | <80ms RTT         | \~70ms          |
| Throughput Test   | UE1        | sst: 1      | >30 Mbps          | \~32 Mbps       |
| Latency Test      | UE2        | sst: 2      | <15ms             | \~12ms          |

---

### **Impact and Efficiency Gains**

By deploying a slice-based 5G network, the project demonstrates how dedicated slices for specific communication goals (latency vs throughput) improve reliability and predictability—essential for V2X use cases where real-time responsiveness is crucial. Vehicle-to-vehicle alert systems (on the latency slice) avoid congestion impacts caused by heavy data streams (handled on the throughput slice).

Running UEs in isolated VMs while maintaining seamless communication with the host-based 5G core and external IoT services also proves the flexibility and scalability of this architecture for research or prototyping in intelligent transportation systems.

---

### **Future Scope**

* Add Mobility support (Handover testing)
* Integrate with a real Qualcomm 5G device for field simulation
* Incorporate QXDM logs for deep packet inspection and diagnostics
* Can have multiple slices to improve security. connectivity and reliability.
* Extend to 5G SA slicing orchestration platforms like OpenSlice


