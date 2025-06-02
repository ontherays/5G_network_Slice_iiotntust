## Checking open5gs Status

### Open5gs
- sudo wireshark

- select interface: lo or uesimtun0

- sctp || udp.port == 8805 || udp.port == 2152

![image](https://github.com/user-attachments/assets/26b8d250-bb0c-4017-b572-f9cd636ae2a7)


<img width="635" alt="image" src="https://github.com/user-attachments/assets/4298412a-001f-4981-abf8-e5a5036400ca" />

### Checking UERANSIM gNB log

- If gNB runs on a separate IP (e.g., 127.0.0.10), filter:

- ip.addr == 127.0.0.10

- ![image](https://github.com/user-attachments/assets/ccd7d7c4-d086-41e1-9d2e-5c05ab9976ac)


<img width="850" alt="image" src="https://github.com/user-attachments/assets/ba82403a-b6d9-46e9-8f13-5783a5db0de8" />

### UE logs

1- Source UE- ping 10.45.0.2  # Ping UE IP from host (should respond)

2- ping 8.8.8.8    # Ping external IP from host (should respond)

![image](https://github.com/user-attachments/assets/72778642-b3ed-4523-9509-1973c15b465c)



![Screenshot 2025-05-30 232207](https://github.com/user-attachments/assets/219f44c2-d755-409b-9c2c-74e6aaa61aec)

### Tesing UE to remote EC2 instance

![image](https://github.com/user-attachments/assets/64b0d3f8-281a-4629-8eba-f3ab06b52568)


<img width="918" alt="image" src="https://github.com/user-attachments/assets/1936db87-5747-414f-9c10-b3a851779eaa" />




