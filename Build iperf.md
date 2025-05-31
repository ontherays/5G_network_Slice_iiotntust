## Building iperf3 and Test

### Install iperf3 

1- sudo apt update

2- sudo apt install -y iperf3

### Latency test

<img width="365" alt="iperf latency" src="https://github.com/user-attachments/assets/8c6aa4a4-8aef-4500-b9c8-88a0bc385432" />

## Test UE ↔ Core (Loopback Core Test)

### Run client using UE's TUN IP 

iperf3 -c 127.0.0.1 -B 10.45.0.2

<img width="482" alt="image" src="https://github.com/user-attachments/assets/e39e5146-6238-46f3-8eb7-35f9e3e823ad" />

### wireshark log

<img width="918" alt="image" src="https://github.com/user-attachments/assets/435a713a-d072-4f71-8d55-fb225f3028fb" />




