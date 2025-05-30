## Checking open5gs Status

### Open5gs
- sudo wireshark

- select interface: lo or uesimtun0

- sctp || udp.port == 8805 || udp.port == 2152


<img width="635" alt="image" src="https://github.com/user-attachments/assets/4298412a-001f-4981-abf8-e5a5036400ca" />

### Checking UERANSIM gNB log

If gNB runs on a separate IP (e.g., 127.0.0.10), filter:

ip.addr == 127.0.0.10

<img width="850" alt="image" src="https://github.com/user-attachments/assets/ba82403a-b6d9-46e9-8f13-5783a5db0de8" />

### UE logs

![Screenshot 2025-05-30 232207](https://github.com/user-attachments/assets/219f44c2-d755-409b-9c2c-74e6aaa61aec)



