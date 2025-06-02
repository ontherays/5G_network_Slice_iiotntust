## Building iperf3 and Test

### Install iperf3 

1- sudo apt update

2- sudo apt install -y iperf3


### Latency and Jitter test

1- ping -I uesimtun0 10.45.0.2

![image](https://github.com/user-attachments/assets/3c9417ab-ef86-4b83-a99b-ab3e6ee2da63)


### Test UE Core (Loopback Core Test)

### Run UE as Client TUN IP 

1- open Terminal - iperf3 -s 127.0.0.1 # Server

2- Open another Terminal iperf3 -c 127.0.0.1 -B 10.45.0.2

![image](https://github.com/user-attachments/assets/bc4d5968-539c-4d24-bb31-cfaca7070b0d)










