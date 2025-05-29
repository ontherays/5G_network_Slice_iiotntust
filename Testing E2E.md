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

