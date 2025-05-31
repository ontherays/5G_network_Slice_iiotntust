# Testing Steps

## 1- Run Open5gs

   $ cd open5gs
   
   $ ./start_open5gs.sh
   
   <img width="368" alt="image" src="https://github.com/user-attachments/assets/042e18f4-499d-42c7-96bf-d5f7913d2784" />


   $ ./status_open5gs
   
   ![image](https://github.com/user-attachments/assets/7922b1bd-91db-4822-997a-3e1b6c1473f2)


## 2- Run gNB

   $ cd UERANSIM
   
   $ sudo ./build/nr-gNB -c config/open5gs-gNB.yaml
   
   ![image](https://github.com/user-attachments/assets/7e9d5533-bfc2-4411-8855-12a66910e6a9)


   Check AMF status:
    $ sudo service open5gs-amfd status

   ![Screenshot 2025-05-01 134728](https://github.com/user-attachments/assets/097891c2-1deb-45df-827e-512dba18e3ef)


## 3-  Run UE

   $ cd UERANSIM
   
   $ sudo ./build/nr-ue -c config/open5gs-ue.yaml
   
   ![Screenshot 2025-05-01 135113](https://github.com/user-attachments/assets/d2abe60c-0b6a-4e8a-998e-9f77468edbda)

   ## UERANSIM_Registartiob_Call Flow

   ![UERANSIM_Registartiob_Call Flow](https://github.com/user-attachments/assets/2f616b98-64c5-4721-9846-ad4c97fec2df)

## Latency and Jitter test

   1- ping -I uesimtun0 10.45.0.2

   <img width="365" alt="iperf latency" src="https://github.com/user-attachments/assets/8c6aa4a4-8aef-4500-b9c8-88a0bc385432" />

## Test UE Core (Loopback Core Test)

### Run UE as Client TUN IP 

   1- open Terminal - iperf3 -s 127.0.0.1 # Server

   2- Open another Terminal iperf3 -c 127.0.0.1 -B 10.45.0.2

   <img width="482" alt="image" src="https://github.com/user-attachments/assets/e39e5146-6238-46f3-8eb7-35f9e3e823ad" />

### wireshark log

<img width="918" alt="image" src="https://github.com/user-attachments/assets/435a713a-d072-4f71-8d55-fb225f3028fb" />

## Run UE as server

   1- open Terminal - iperf3 -s 10.45.0.2 # Server

   2- Open another Terminal iperf3 -c 10.45.0.2 -B 127.0.0.1


   ![Screenshot 2025-05-31 112404](https://github.com/user-attachments/assets/5788d467-f9fa-4bf3-94cd-bb9e12447d4e)


   <img width="917" alt="Screenshot 2025-05-31 112716" src="https://github.com/user-attachments/assets/9aa6aea8-79d1-438f-8905-29f6d60a0c53" />

## Verify UE to IOT devices connection which will emulate UE(Vehicle) and IOT as other things(Everrything)

   1- Crete EC2 instance on AWS

   2- Set inbound traffic rules

     Edit inbound rules:
  
       Type: Custom TCP
    
       Port range: 5201
    
       Source: Your lab PC's public IP (you can find it via https://ifconfig.me)
    
   3- SSH into EC2:
   
   ssh -i EC2Key_Pair.pem ubuntu@3.84.42.160

   4- iperf3 -s

   <img width="485" alt="image" src="https://github.com/user-attachments/assets/1c378583-acba-4845-b71b-1563f65d8dae" />


   5- iperf3 -c 3.84.42.160 -t 30 -i 5

   <img width="484" alt="image" src="https://github.com/user-attachments/assets/1defc506-2526-4e34-ae36-5b43c0ad3b15" />

## Vehicle(UE) to Vehicle(UE)

 - Configuration
   
    1- amf.yaml
 
       ~~~
        - smf:
         pdn:
        - addr: 10.45.0.1/16
          dnn: internet
         ~~~
     2- login webui - Enter subscriber
            imsi-001010000000002

     3- Create open5gs-ue2.yaml in UERANSIM with new subscriber.

     4- Run Open5gs

     5- Run open5gs-gNB.yaml

  6- Run open5gs-ue.yaml and open5gs-ue2.yaml in seperate windows.

   UE 1 - uesimtun0, 10.45.0.8

   UE 2 - uesimtun0, 10.45.0.9

   <img width="1049" alt="Screenshot 2025-05-31 192529" src="https://github.com/user-attachments/assets/c4bcab9b-2239-46e1-b581-e5714470da0a" />

   ### Verify both UE can ping each Other-

   <img width="505" alt="Screenshot 2025-05-31 195808" src="https://github.com/user-attachments/assets/47378af8-5cb5-455f-bc41-ae3068b5b181" />

   #### Iperf from UE 1 to UE 2.

   - iperf3 -c 10.45.0.9 -B 10.45.0.8 -t 10

     <img width="515" alt="Screenshot 2025-05-31 200643" src="https://github.com/user-attachments/assets/99182b30-22d4-4769-97fd-9053c6097eb3" />

   #### Iperf from UE 2 to UE 1.

  - iperf3 -c 10.45.0.8 -B 10.45.0.9 -t 10

    <img width="512" alt="Screenshot 2025-05-31 201456" src="https://github.com/user-attachments/assets/00676e01-6557-4a68-86bb-2168179aafe7" />

