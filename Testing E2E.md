# Testing Steps

## 1- Run Open5gs

   $ cd open5gs
   
   $ start-check-open5gs.sh

   ![image](https://github.com/user-attachments/assets/7039b3ea-8224-4bc0-9456-d786706e37b8)
   


   $ amf log

   ![image](https://github.com/user-attachments/assets/12eff0dd-ac0c-450d-9c3f-163ab81a4df5)

   ![image](https://github.com/user-attachments/assets/6bfbbd82-c268-44bc-a28b-332a196cb2c0)

   


## 2- Run gNB

   $ cd UERANSIM
   
   $ sudo ./build/nr-gnb -c config/open5gs-gnb.yaml
   
   ![image](https://github.com/user-attachments/assets/9e6333e2-988b-4507-9887-1186efc9784f)



   Check AMF status:
    $ sudo service open5gs-amfd status

   ![image](https://github.com/user-attachments/assets/f93b6c50-b714-4aa4-8088-3cc766884a1c)



## 3-  Run UE

   $ cd UERANSIM
   
   UE1- uesimtun0, 10.45.0.3
   
   $ sudo ./build/nr-ue -c config/open5gs-ue.yaml
   
   ![image](https://github.com/user-attachments/assets/b4907c41-80f5-41f7-a91a-ae55484c7c99)
   
   $ sudo ./build/nr-ue -c config/open5gs-ue2.yaml

   UE2 - uesimtun1, 10.45.0.4

   ![image](https://github.com/user-attachments/assets/5c9dd830-a0a2-4946-8bf3-d37ab720d3f9)



   ## UERANSIM_Registartiob_Call Flow

   ![UERANSIM_Registartiob_Call Flow](https://github.com/user-attachments/assets/2f616b98-64c5-4721-9846-ad4c97fec2df)

## Latency and Jitter test

   1- ping -I uesimtun0 10.45.0.2

   ![image](https://github.com/user-attachments/assets/d8fc0194-8cc5-473d-9b5d-0ee85a8b4075)


## Test UE Core (Loopback Core Test)

### Run UE as Client TUN IP 

   1- open Terminal - iperf3 -s 127.0.0.1 # Server

   2- Open another Terminal iperf3 -c 127.0.0.1 -B 10.45.0.2

   ![image](https://github.com/user-attachments/assets/ccda3fda-6c14-4690-a36e-c31e09c490d5)



### wireshark log

![image](https://github.com/user-attachments/assets/e9200505-215e-412a-be63-f214aba6e383)


## Run UE as server

   1- open Terminal - iperf3 -s 10.45.0.2 # Server

   2- Open another Terminal iperf3 -c 10.45.0.2 -B 127.0.0.1

    ![image](https://github.com/user-attachments/assets/296da8eb-8bd2-4e73-8865-8b13cf8c6f27)

 ### wireshark log

    ![image](https://github.com/user-attachments/assets/e33c58ef-d6c6-482b-8716-d65cfd0710e9)



   ![Screenshot 2025-05-31 112404](https://github.com/user-attachments/assets/5788d467-f9fa-4bf3-94cd-bb9e12447d4e)


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

   ![image](https://github.com/user-attachments/assets/d927ef47-301b-4197-ba99-23c1236e86e2)


   5- iperf3 -c 3.84.42.160 -t 30 -i 5

   ![image](https://github.com/user-attachments/assets/0c49a053-2137-44e1-8220-28a657f24451)


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

   ![image](https://github.com/user-attachments/assets/c8577ee6-b2af-4398-b0ad-e3f503f81f37)


   ### Verify both UE can ping each Other-

   ![image](https://github.com/user-attachments/assets/4157d1a6-cfe0-47ee-9346-c084de3aa0b3)


   #### Iperf from UE 1 to UE 2.

   - iperf3 -c 10.45.0.9 -B 10.45.0.8 -t 10

      - ![image](https://github.com/user-attachments/assets/ab76c57c-09bb-41aa-8d8c-c32b87f0a8d2)


   #### Iperf from UE 2 to UE 1.

  - iperf3 -c 10.45.0.8 -B 10.45.0.9 -t 10

     - ![image](https://github.com/user-attachments/assets/22a7c1cd-d5fc-42dd-98b7-bf6829027173)
 

