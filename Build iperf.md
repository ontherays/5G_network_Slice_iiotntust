## Building iperf3 and Test

### Install iperf3 

1- sudo apt update

2- sudo apt install -y iperf3

### Latency test

<img width="365" alt="iperf latency" src="https://github.com/user-attachments/assets/8c6aa4a4-8aef-4500-b9c8-88a0bc385432" />

## Test UE ↔ Core (Loopback Core Test)

### Run UE as Client TUN IP 

1- open Terminal - iperf3 -s 127.0.0.1 # Server

2- Open another Terminal iperf3 -c 127.0.0.1 -B 10.45.0.2

<img width="482" alt="image" src="https://github.com/user-attachments/assets/e39e5146-6238-46f3-8eb7-35f9e3e823ad" />

### wireshark log

<img width="918" alt="image" src="https://github.com/user-attachments/assets/435a713a-d072-4f71-8d55-fb225f3028fb" />

### Run UE as server

1- open Terminal - iperf3 -s 10.45.0.2 # Server

2- Open another Terminal iperf3 -c 10.45.0.2 -B 127.0.0.1


![Screenshot 2025-05-31 112404](https://github.com/user-attachments/assets/5788d467-f9fa-4bf3-94cd-bb9e12447d4e)


<img width="917" alt="Screenshot 2025-05-31 112716" src="https://github.com/user-attachments/assets/9aa6aea8-79d1-438f-8905-29f6d60a0c53" />

### Verify testing to UE to IOT devices which will emulate UE as Vehicle and IOT as other things(Everrything)

1- Crete EC2 instance on AWS

2- Set inbound traffic rules

  Edit inbound rules:
  
    Type: Custom TCP
    
    Port range: 5201
    
    Source: Your lab PC's public IP (you can find it via https://ifconfig.me)
    
3- SSH into EC2:
ssh -i EC2Key_Pair.pem ubuntu@3.84.42.160

4- iperf3 -s

![Screenshot 2025-05-31 135647](https://github.com/user-attachments/assets/56026855-f5a1-4428-8c18-9cd697851dbf)

5- iperf3 -c 3.84.42.160 -t 30 -i 5

<img width="483" alt="image" src="https://github.com/user-attachments/assets/d08d2170-7286-41c5-bcd3-6ccb42d68207" />



