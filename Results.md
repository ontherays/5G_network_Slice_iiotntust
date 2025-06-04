## **Results and Achievements**

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
