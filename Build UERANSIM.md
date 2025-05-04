## Build UERANSIM

**Getting the UERANSIM**
   $ cd ~
   
   $ git clone https://github.com/aligungr/UERANSIM

**Dependencies**
   $ sudo apt update
   
   $ sudo apt upgrade

   Then here's the list of dependencies: (Built-in dependencies shipped with Ubuntu are not listed herein.)

   $ sudo apt install make
   
   $ sudo apt install gcc
   
   $ sudo apt install g++
   
   $ sudo apt install libsctp-dev lksctp-tools
   
   $ sudo apt install iproute2
   
   $ sudo snap install cmake --classic



**Building**

command for building:

cd ~/UERANSIM

make

## Configuration

### UERANSIM Configuration for Open5GS

### gNB Configuration (`gnb.yaml`)

| Parameter               | Value                     | Description                                  |
|-------------------------|---------------------------|----------------------------------------------|
| `mcc`                   | `001`                     | Mobile Country Code (match Open5GS PLMN)     |
| `mnc`                   | `01`                      | Mobile Network Code (match Open5GS PLMN)     |
| `nci`                   | `0x123456`                | gNB ID (unique identifier)                  |
| `idLength`             | `32`                      | ID length                                   |
| `tac`                   | `1`                       | Tracking Area Code                          |
| `linkIp`                | `127.0.0.1`               | gNB local IP address                       |
| `ngapIp`                | `127.0.0.1`               | gNB local IP for NGAP                       |
| `gtpIp`                 | `127.0.0.1`               | gNB local IP for GTP                        |
| `amfConfigs:`          |                           | AMF configuration                           |
| `- address:`           | `127.0.0.5`               | Open5GS AMF IP address                      |
| `port:`                | `38412`                   | Open5GS AMF port (default)                  |
| `gnbSearchList:`       |                           | gNB search list (empty for local)           |

### UE Configuration (`ue.yaml`)

| Parameter               | Value                     | Description                                  |
|-------------------------|---------------------------|----------------------------------------------|
| `supi`                  | `imsi-001010000000001`    | UE IMSI (match Open5GS subscriber)          |
| `mcc`                   | `001`                     | Mobile Country Code                         |
| `mnc`                   | `01`                      | Mobile Network Code                         |
| `key`                   | `465B5CE8B199B49FAA5F0A2EE238A6BC` | Authentication key (match Open5GS) |
| `op`                    | `E8ED289DEBA952E4283B54E88E6183CA` | Operator key (match Open5GS)       |
| `opType`                | `OPC`                     | Operator code type                          |
| `amf`                   | `8000`                    | Authentication Management Field             |
| `sqn`                   | `000000000020`            | Sequence number                             |
| `dnn`                   | `internet`                | Data Network Name (match Open5GS)           |
| `gnbSearchList:`       | `[127.0.0.5]`             | gNB IP address to connect                   |
| `uectl:`               |                           | UE control settings                         |
| `- addr:`              | `127.0.0.100`             | UE TUN interface IP                         |
| `- addr:`              | `10.45.0.2`               | UE internal IP                              |

#### Important Notes

1. Ensure Open5GS is properly configured with matching:
   - PLMN (MCC=001, MNC=01)
   - Subscriber IMSI and authentication keys
   - AMF binding to 127.0.0.1:38412

2. For VSCode debugging:
   ```bash
   # Run gNB
   ./build/nr-gnb -c config/open5gsgnb.yaml

   # Run UE
   ./build/nr-ue -c config/open5gsue.yaml
   ```

3. For network connectivity, enable IP forwarding and NAT if needed.

Save this as `ueransim_config.md` in your VSCode project. Adjust IP addresses and values to match your specific Open5GS configuration. The key parameters that must match between UERANSIM and Open5GS are:
- MCC/MNC
- IMSI/SUPI
- Authentication keys (K/OP/OPc)
- AMF IP/port
- DNN name
