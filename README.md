## 5G_network_Slice_iiotntust
This project aimed to design, simulate, and test a 5G network slicing framework for Vehicle-to-Everything (V2X) communications, ensuring ultra-low latency, high reliability, and efficient resource allocation.

# System Architecture

![V2X_Open5gs](https://github.com/user-attachments/assets/8ae2d196-74e4-4587-990a-c5274b580bd0)

### Key Updates & Annotations

  IP Addressing

    UE (Vehicle1): 10.45.0.1/24
    
    UE (Vehicle2): 10.45.0.2/24 (Fixed typo from your input: both UEs can’t have same IP)
    
    UPF (Open5GS): 10.45.0.100/24 (Gateway for UEs)

  Icons for Clarity

    🏢: AWS IoT Core
  
    🖥️: V2X Application Server
  
    🗃️: Database (DynamoDB/Timestream)
  
    📊: Analytics (QuickSight/Grafana)
  
    📡: AMF (Control Plane)
  
    🔌: SMF (Session Management)
  
    🚀: UPF (User Plane)
  
    📶: gNodeB (RAN)
  
    🚗/🚙: UEs (Vehicles)
  
    🔄: V2V Sidelink Communication

Interfaces Labeled

    N1/N2: NGAP (UE ↔ AMF, gNodeB ↔ AMF)

    N3: GTP-U (gNodeB ↔ UPF)

    PC5: Sidelink (Direct V2V)

