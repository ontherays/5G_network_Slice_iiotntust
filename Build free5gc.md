## Installation

### [Free5GC] Installation Guide
1. Requirement
1.1 Check Kernel Version
1.2 Install Golang
1.3 Install Dependency for Control Plane Element
1.4 Install Dependency for User Plane Element
2. Install Network Function Element
2.1 Install GTP
2.2 Install All the Element
3. Install Webconsole
3.1 Install Nodejs and Yarn Package
3.2 Install Webconsole
4. Setup Config
4.1 Check your Interface Card
4.2 Host Network Setting
4.3 Edit AMF Config
4.4 Edit SMF Config
4.5 Edit UPF Config
5. Run !
5.2 Execute Free5GC
5.3 Execute Web Console
5.4 Check Port
6. Note
6.1 Restart

1. Requirement
Hardware Requirement:

CPU: 4 vCPU
RAM: 8 GB
Disk: 40 GB
Software Requirement:

OS Kernel: 5.4.x => ubuntu 18.04
golang: 1.14.4
1.1 Check Kernel Version

# uname -r
5.4.0-126-generic
1.2 Install Golang












# Download go 1.14
wget https://dl.google.com/go/go1.14.4.linux-amd64.tar.gz

# Install
sudo tar -C /usr/local -zxvf go1.14.4.linux-amd64.tar.gz
mkdir -p ~/go/{bin,pkg,src}

# Set ENV
echo 'export GOPATH=$HOME/go' >> ~/.bashrc
echo 'export GOROOT=/usr/local/go' >> ~/.bashrc
echo 'export PATH=$PATH:$GOPATH/bin:$GOROOT/bin' >> ~/.bashrc
echo 'export GO111MODULE=auto' >> ~/.bashrc
source ~/.bashrc
Check:


# go version
go version go1.14.4 linux/amd64
1.3 Install Dependency for Control Plane Element


sudo apt -y update
sudo apt -y install mongodb wget git net-tools curl
sudo systemctl start mongodb
1.4 Install Dependency for User Plane Element
sudo apt -y install git gcc g++ cmake autoconf libtool pkg-config libmnl-dev libyaml-dev
2. Install Network Function Element
2.1 Install GTP




cd ~
git clone https://github.com/free5gc/gtp5g.git
cd ~/gtp5g
make
sudo make install
2.2 Install All the Element
Clone source code:


cd ~
git clone --recursive -b v3.2.1 -j `nproc` https://github.com/free5gc/free5gc.git
Compile and Install:


cd ~/free5gc
make
3. Install Webconsole
3.1 Install Nodejs and Yarn Package







cd ~
sudo apt remove cmdtest
sudo apt remove yarn
curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -
sudo apt-get update
sudo apt-get install -y nodejs yarn
3.2 Install Webconsole

cd ~/free5gc
make webconsole
4. Setup Config
4.1 Check your Interface Card
Find Gateway Interface:
In this case, it is ens33









# route -n 

Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         192.168.8.9     0.0.0.0         UG    100    0        0 ens33
10.100.200.0    0.0.0.0         255.255.255.0   U     0      0        0 br-free5gc
169.254.0.0     0.0.0.0         255.255.0.0     U     1000   0        0 ens33
172.17.0.0      0.0.0.0         255.255.0.0     U     0      0        0 docker0
192.168.8.0     0.0.0.0         255.255.255.0   U     100    0        0 ens33
Find IP Address:
IP is 192.168.8.32













# ifconfig

ens33: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.8.32  netmask 255.255.255.0  broadcast 192.168.8.255
        inet6 fe80::97ee:fa0:b157:c4df  prefixlen 64  scopeid 0x20<link>
        inet6 fd90:f942:f30c:0:154b:cbf7:a6a3:4257  prefixlen 64  scopeid 0x0<global>
        inet6 fd90:f942:f30c:0:bd96:55cf:f843:319a  prefixlen 64  scopeid 0x0<global>
        inet6 fd90:f942:f30c::ca7  prefixlen 128  scopeid 0x0<global>
        ether 00:0c:29:1f:b1:1b  txqueuelen 1000  (Ethernet)
        RX packets 1244837  bytes 1830588982 (1.8 GB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 140181  bytes 12973947 (12.9 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
4.2 Host Network Setting
Check Interface:
Note: <dn_interface> should be replace to ens33









# Enable IP forward
sudo sysctl -w net.ipv4.ip_forward=1

# Setting NAT
sudo iptables -t nat -A POSTROUTING -o <dn_interface> -j MASQUERADE
sudo iptables -A FORWARD -p tcp -m tcp --tcp-flags SYN,RST SYN -j TCPMSS --set-mss 1400

# Stop Firewall
sudo systemctl stop ufw
4.3 Edit AMF Config
vim ~/free5gc/config/amfcfg.yaml



configuration:
  amfName: AMF # the name of this AMF
  ngapIpList:  # the IP list of N2 interfaces on this AMF
    - 192.168.8.32 # Default is 127.0.0.18
4.4 Edit SMF Config
vim ~/free5gc/config/smfcfg.yaml











configuration:
  ...
  userplaneInformation:
    ...
    upNodes:
      ...
      UPF:
        ...
        interfaces:
          - interfaceType: N3 # the type of the interface (N3 or N9)
            endpoints: # the IP address of this N3/N9 interface on this UPF
              - 192.168.8.32 # Default is 127.0.0.8
Note: Change UPF IP carefully! If your Free5GC is deployed inside a network, you should use the IP at the same network as gNB.

4.5 Edit UPF Config
vim ~/free5gc/config/upfcfg.yaml









gtpu:
  forwarder: gtp5g
  # The IP list of the N3/N9 interfaces on this UPF
  # If there are multiple connection, set addr to 0.0.0.0 or list all the addresses
  ifList:
    - addr: 192.168.8.32 # Default is 127.0.0.8
      type: N3
      # name: upf.5gc.nctu.me
      # ifname: gtpif

5. Run !
5.2 Execute Free5GC
Command:


cd ~/free5gc/
./run.sh
5.3 Execute Web Console
Command:


cd ~/free5gc/webconsole
go run server.go
Web Console Information:

URL: http://127.0.0.1:5000 or http://<Free5GC_IP>:5000
Default User: admin
Default Password: free5gc
5.4 Check Port




# Check for SCTP
netstat -ln | grep sctp

# Check for TCP and UDP
netstat -tulpn
6. Note
6.1 Restart
If restart the system, you need to rebuild the gtp5g again.
