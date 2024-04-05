# 5G-ovs-integration with MEC & slicing capability
This experiment has been performed by 5G use case lab (5GUCL) IDRBT.
Requirements: USRP B210, PC with 16GB RAM and a hexacore processor 
### Prerequisites
```Docker```: https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04
```Docker-compose``` : https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-compose-on-ubuntu-20-04
Link to the blog: https://5guclidrbt.blogspot.com/2023/09/5g-slicing-for-secure-banking-edge.htmlF
<!--
![5gedge drawio](https://github.com/5g-ucl-idrbt/5G-ovs-integration/assets/46273637/55aef223-125f-4115-ac3d-8fb8d261ef38)
-->
![5gedge drawio(1)](https://github.com/5g-ucl-idrbt/5G-ovs-integration/assets/46273637/d3b1d366-489c-4edb-9261-9ec36f780e1a)



## Refer these links to setup core in VM and Physical gNB
In developv4 branch we are using physical devices to test the setup core version v1.5.1, while developv2 and developv3 use simulated environment. 
In the develop branch we are using version 1.4.0 of OAI core. Here in developv2 and developv3 we are using version 1.5.1 from master branch.
<!--
for core: ```https://gitlab.eurecom.fr/oai/cn5g/oai-cn5g-fed``` 

```
cd
git clone https://gitlab.eurecom.fr/oai/cn5g/oai-cn5g-fed.git
```
## Adding files to oai-setup
Clone the repo and paste the docker-compose file and createLink.sh file in the oai-cn5g-fed/docker-compose path
```
cd
git clone https://github.com/abhic137/5G-ovs-slicing.git
cd 5G-ovs-slicing
git branch -a
git checkout developv4
cd 5G-ovs-slicing/docker-compose
cp run.sh createLink.sh docker-compose-basic-nrf-ovs.yaml ~/oai-cn5g-fed/docker-compose
cp oai_db3.sql ~/oai-cn5g-fed/docker-compose/database

``` 
-->

## Clone the repo
```
cd
git clone https://github.com/5g-ucl-idrbt/5G-ovs-slicing.git
git branch -a
git checkout developv4
```
## Build the required images

for RYU
```
 sudo docker build -f Dockerfile.Ryu -t osrg/ryu:latest --network host .
```
for SPGWU
```
sudo docker build -f Dockerfile.SPGWU -t oaisoftwarealliance/oai-spgwu-tiny:v1.5.1  --network host .  
```
for UBUNTU
```
sudo docker build -f Dockerfile.Ubuntu -t ubuntu:latest --network host . 
```
for RTMP Server & Speedtest server
```
cd component
sudo ./mergeAndCreate.sh
```

## Pull the required images
```
sudo docker pull openvswitch/ovs:2.11.2_debian
sudo docker tag openvswitch/ovs:2.11.2_debian openvswitch/ovs:latest
```
# Running the Core
```
cd 5G-ovs-slicing/docker-compose
sudo docker compose -f docker-compose-basic-nrf-ovs.yaml up -d
sudo docker ps -a
```
```OR```
If you want to use the speed testerserver as well as the rtmp server you can use these commands
```
cd 5G-ovs-slicing/docker-compose
sudo docker compose -f docker-compose-basic-nrf-ovs-streaming.yaml up -d
sudo docker ps -a
```
```OR```
For Banking app deployment

Click [Banking Secure Slice](#secured-banking-slice-demo)

## Run the script file to create bridge, connections and to add IPs & routes 
```
cd oai-cn5g-fed/docker-compose
chmod +x run.sh
sudo ./run.sh
```
<!--
## For getting internet connection in the UE
```
sudo docker exec -it router bash
ifconfig
iptables -A FORWARD -i <dcp_INT> -o eth0 -j ACCEPT
iptables -A FORWARD -i eth0 -o <dcp_INT> -m state --state RELATED,ESTABLISHED -j ACCEPT
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

```
-->
<!--
   * Create Links
```
sudo bash createLink.sh oai-spgwu s1
sudo bash createLink.sh server s1
sudo bash createLink.sh router s1
```
   * Create bridge in s1
```
sudo docker exec s1 ovs-vsctl add-br br0
sudo ip netns exec s1 ifconfig -a | grep -E "dcp.*"| awk -F':' '{print $1}'   ## List interface names; Use Outputs as inputs of next CMD separately
sudo docker exec s1 ovs-vsctl add-port br0 <INTF1>
sudo docker exec s1 ovs-vsctl add-port br0 <INTF2>
sudo docker exec s1 ovs-vsctl add-port br0 <INTF3>
sudo docker exec s1 ovs-vsctl set-fail-mode br0 secure
```
   * Add IP to oai-spgwu and tomcat
```
C1_IF=$(sudo ip netns exec oai-spgwu ifconfig -a | grep -E "dcp.*"| awk -F':' '{print $1}')
sudo ip netns exec oai-spgwu ip a add 10.0.0.1/24 dev ${C1_IF}
C2_IF=$(sudo ip netns exec tomcat ifconfig -a | grep -E "dcp.*"| awk -F':' '{print $1}')
sudo ip netns exec server ip a add 10.0.0.2/24 dev ${C2_IF}
C3_IF=$(sudo ip netns exec router ifconfig -a | grep -E "dcp.*"| awk -F':' '{print $1}')
sudo ip netns exec router ip a add 10.0.0.3/24 dev ${C3_IF}

sudo ip netns exec oai-spgwu ip r add 10.0.0.3/32 via 0.0.0.0 dev ${C1_IF}
sudo ip netns exec oai-spgwu ip r add 10.0.0.2/32 via 0.0.0.0 dev ${C1_IF}
sudo ip netns exec server ip r add 10.0.0.1/32 via 0.0.0.0 dev ${C2_IF}
sudo ip netns exec router ip r add 10.0.0.1/32 via 0.0.0.0 dev ${C3_IF}
```

* Add controller to the ovs
```
sudo docker exec s1 ovs-vsctl set-controller br0 tcp:172.18.0.4:6653
```
-->
<!--
## Running the Ryu code (For simple switch functionality)
* Inside the ryu container
  Here we are running a simple switch program, we can run any custom program in the same manner.
```
sudo docker exec ryu ryu-manager --observe-links ryu/ryu/app/simple_switch.py 
```
OR
```
sudo docker exec -it ryu bash
cd ryu/ryu/app
ryu-manager --observe-links simple_switch.py 
```
-->
# Slicing
## For running the slicing code
```
sudo docker exec ryu ryu-manager --observe-links ryu/ryu/app/ryucode.py
```

<!--
```
sudo docker exec -it ryu bash
cd ryu/ryu/app
```
Before running the ryu code, change the IP address of the server and router in the code. And also change the MAC addresses accordingly. to take IP and mac address of the server,router & oai-spgwu
On line no 105 change MAC address of server(10.0.0.2) On line no 106 change MAC address of server(10.0.0.2) On line no 124 change MAC address of router(10.0.0.3) On line no 125 change MAC address of oai-spgwu

in the ryu docker 
```
nano ryucode.py

```
In the other terminal copy the MACs of the server,router and spgwu and paste it in the ryucode.py
```
sudo docker exec server ifconfig
```
```
sudo docker exec router ifconfig
```
```
sudo docker exec oai-spgwu ifconfig
```
In order to run the RYU code
```
ryu-manager --observe-links ryucode.py 

```
-->

<!--
## In spgwu 
```
sudo docker exec -it oai-spgwu bash
```
```
apt update
apt install -y iputils-ping
apt install -y tcpdump
apt install -y iproute2
apt install -y iptables
```
```
sysctl net.ipv4.ip_forward=1
iptables -P FORWARD ACCEPT
ip route del default via 192.168.70.129 dev eth0
ifconfig #copy the port name starting with dcp and paste it in the <dev_name> in nextline
ip route add default via 10.0.0.3 dev <dev_name> #this will help UE to reach the internet through the router pc 
 
```
## Inside the server docker
```
sudo docker exec -it server bash
```
```
apt update
apt install -y iputils-ping
apt install -y tcpdump
apt install -y iproute2
apt install -y iptables
apt install -y net-tools
```
```
ifconfig # copy the dcp dev name
ip route add 12.1.1.0/24 via 10.0.0.1 dev <DEV_NAME>

```
## Inside the router docker
```
sudo docker exec -it router bash
```
```
apt update
apt install -y iputils-ping
apt install -y tcpdump
apt install -y iproute2
apt install -y iptables
apt install -y net-tools
```
```
ifconfig # copy the dcp dev name
ip route add 12.1.1.0/24 via 10.0.0.1 dev <DEV_NAME>

```
-->

## Test to check if the ovs is properly configure
In a new terminal 
```
sudo docker exec oai-spgwu ping -c3 10.0.0.2
sudo docker exec oai-spgwu ping -c3 10.0.0.3
sudo docker exec server ping -c3 10.0.0.1
sudo docker exec router ping -c3 10.0.0.1
```
## Hosting a simple python server in server docker
In a new terminal
```
sudo docker exec server python3 -m http.server 9999
```
```OR```
```
sudo docker exec -it server bash

python3 -m http.server 9999


```
## Commmands to be executed in Core VM in order to connect to the gNB
```
sudo sysctl net.ipv4.ip_forward=1
sudo iptables -P FORWARD ACCEPT
sudo ip route add 192.168.71.194 via <GNB Baremetal IP>
sudo ip route add 12.1.1.0/24 via 192.168.70.134 # Forward packets to Mobiles from external sources
```
To check if the devices are connected to core follow the AMF logs
```
sudo docker logs --follow oai-amf
```
# Setting up gNB
Clone this repo  and follow the instructions ref: https://github.com/5g-ucl-idrbt/oai-gnodeb-b210
## Commands to be executed in gNB

```
sudo sysctl net.ipv4.ip_forward=1
sudo iptables -P FORWARD ACCEPT
sudo ip route add 192.168.70.128/26 via <Bridge IP of Core VM>
```
```
cd ci-scripts/yaml_files/sa_b200_gnb/
sudo docker-compose up -d
```
```
sudo docker exec -it sa-b200-gnb bash
```
```
bash bin/entrypoint.sh
/opt/oai-gnb/bin/nr-softmodem -O /opt/oai-gnb/etc/gnb.conf $USE_ADDITIONAL_OPTIONS
```


<!--
# Testing with GNBSIM instead of physical USRP and UE

# For attaching 1 gNB and 1 UE
```
cd oai-cn5g-fed/docker-compose
sudo docker-compose -f docker-compose-gnbsim.yaml up -d gnbsim
sudo docker ps -a
```
For getting the IP of UE
```
sudo docker logs gnbsim
```

## Ping tests and curl tests 
```
sudo docker exec -it gnbsim bash
```
```
apt update
apt install -y curl
```
Here 12.1.1.2 is the ip of the UE (we can get it by looking into amf logs or in gnbsim logs)
```
#ping -I 12.1.1.2 8.8.8.8
ping -I 12.1.1.2 10.0.0.1
ping -I 12.1.1.2 10.0.0.2
ping -I 12.1.1.2 10.0.0.3

```
### checking connectivity with ther tomcat server (Python server)
```
curl --interface 12.1.1.2 http://10.0.0.2:8888
```
<!--### checking the connectivity to the server (tomcat)
```
curl --interface 12.1.1.2 http://192.168.150.115:8888

```
-->
<!--
# For attaching 2 gnbs and 2 Ues respectively
```
sudo docker-compose -f docker-compose-gnbsim.yaml up -d gnbsim gnbsim2
sudo docker ps -a
```
To know the Ip of the UEs

```
sudo docker logs gnbsim
sudo docker logs gnbsim2
```
-->

Ping tests to perform in UE 
```
ping 8.8.8.8
```
```
ping 10.0.0.1
ping 10.0.0.2
ping 10.0.0.3
```
<!--
## To do the network slicing
To update the mac_to_port dictionary, you need to ping the server and router from the UE.
```
ping 10.0.0.2
```
```
ping 10.0.0.3
```
IF you want to see what is being saved in the dictionary, you can create a json file named as mac_to_port in the same folder you are running ryu controller, and if not needed, remove the line from the code where it is being saved in json file(i.e., line no 27 & 28)
-->

## To verify that network slicing is working
<!--
Run simple python server on server.
NOTE: Always run the python server on the port 9999 (according to the RYU code)
```
sudo docker exec server python -m http.server 9999
```
Run simple python server on router
```
sudo docker exec router python -m http.server 9988
```

Now if you will do the wget command by the ip of router but the tcp_port on which the server is running, it will be replied by server and not router. You can also verify it on the terminal, from where you got the reply.
To do the wget command. Run following command in gnbsim
```
wget --bind-address= <UE_ip_address> <router_ip_address>:9999
```
And if we give some other tcp_port, it will be replied by router
```
wget --bind-address= <UE_ip_address> <router_ip_address>:9988
```
So, you can observe in the terminal(router & server) that even if the ip was same but was answered by differenrt systems.

-->
In the UE open the terminal (termux app) and use the command wget to reach server. But here, we are performing application based slicing we will "wget" the server with the IP of the router which goes towards the internet but not with the actual IP of the server.
```
wget 10.0.0.3:9999 #IP of the router
```
Here we have used the Ip of the router,but the port number is ```9999```. ie., if the UE is trying to reach the internet via the port 9999 it can communicate with the server.
we can observe the logs in the tab where we ran the ```sudo docker exec server python3 -m http.server 9999``` command. By looking at these logs we can conclude that the UE reached the server ie., 10.0.0.2.






## To verify that the UE is going through the router towards the internet
```
sudo docker exec -it router bash
ifconfig 
tcpdump -i <interface_name> #interface starting with dcp 
```
## To verify that the UE is reaching the server
```
sudo docker exec -it server bash
ifconfig
tcpdump -i <interface_name> #interface starting with dcp
```
# For running the speed test in UE
Go to a browser in the UE and type the following ip
```
http://192.168.70.140:3000
```
# For running and viewing the rtmp server stream
to view the stream go to your browser in a pc
```
http://192.168.70.141:9080/players/hls.html

```
```OR```
```
http://<CORE_VM IP>:9080/players/hls.html
```
```OR```
```
http://localhost:9080/players/hls.html
```

in mobile phone you have to use astra app
add this as rtmp server
```
192.168.70.141:1935/live

```
key is ```test```
# For stopping the processes
For shutting down gNB
```
sudo docker-compose down
```
For shutting down the core
```
sudo docker compose -f docker-compose-basic-nrf-ovs.yaml down
```
```OR```
```
sudo docker compose -f docker-compose-basic-nrf-ovs-streaming.yaml down
```
# ----------------------------------------------------------
#            Secured Banking Slice Demo
# ----------------------------------------------------------
## Prerequisites
Before you run for your personalized requirement you have to change : 
- the port number as well as IP addreses in the RYU code. The path is ```5G-ovs-integration/docker-compose/ryuctrlr
/automac_UEbind.py```
Change the UE Ip accordingly which you want in the slice & change the port according to the servers hosted port
```
   Line 80: if (pkt.get_protocol(tcp.tcp) and pkt.get_protocol(tcp.tcp).dst_port == 9999 and pkt.get_protocol(ipv4.ipv4).src=="12.1.1.2"):     #### change the UE Ip accordingly which you want in the slice & change the port according to the servers hosted port ####
```
Change the IP of the server (you also have to change the ip in the run.sh file)
```
Line 91: parser.OFPActionSetField(ipv4_dst="10.0.0.2"),    ### change the IP of the server (you also have to change the ip in the run.sh file) ###
```
Change the port according to the servers hosted port
```
Line 98: elif (pkt.get_protocol(tcp.tcp) and pkt.get_protocol(tcp.tcp).src_port == 9999): ### change the port according to the servers hosted port ###
```
Change the IP of the router (you also have to change the ip in the run.sh file)
```
Line 108: parser.OFPActionSetField(ipv4_src="10.0.0.3"),  ### change the IP of the router (you also have to change the ip in the run.sh file) ###
```
Change the port according to the servers hosted port
```
Line 115: elif (pkt.get_protocol(tcp.tcp) and pkt.get_protocol(tcp.tcp).src_port != 9999 and pkt.get_protocol(tcp.tcp).dst_port != 9999):   ### change the port according to the servers hosted port ###
```
<!--
![Screenshot 2023-08-30 195724](https://github.com/5g-ucl-idrbt/5G-ovs-integration/assets/46273637/ff4d5d92-df84-4a16-9e30-1c18e45507d7)

![Screenshot 2023-08-30 200439](https://github.com/5g-ucl-idrbt/5G-ovs-integration/assets/46273637/27b785a3-4306-4425-b4a1-05428d9fcc67)


![Screenshot 2023-08-30 200500](https://github.com/5g-ucl-idrbt/5G-ovs-integration/assets/46273637/63249daf-773a-4d0e-91dc-d2da6f2efb5a)

-->
Make sure you have built the banking-app image using the docker file present in the ```/dockerfiles``` folder
- run the scenario
```
cd 5G-ovs-integration/docker-compose
sudo docker compose -f docker-compose-slicing-bank-nrf.yaml up -d

```
- Run the slicing setup script
```
cd 5G-ovs-integration//docker-compose
chmod +x run.sh
sudo ./run.sh
```
- Run the slicing code in the RYU controller
```
sudo docker exec ryu ryu-manager --observe-links ryu/ryu/app/ryucode.py
```
- In a new tab observe the AMF logs To check if the devices are connected to core 
```
sudo docker logs --follow oai-amf
```
- Commmands to be executed in Core VM in order to connect to the gNB
```
sudo sysctl net.ipv4.ip_forward=1
sudo iptables -P FORWARD ACCEPT
sudo ip route add 192.168.71.194 via <GNB Baremetal IP>
sudo ip route add 12.1.1.0/24 via 192.168.70.134 # Forward packets to Mobiles from external sources
```

- Setting up gNB in a diffrent PC
Clone this repo  and follow the instructions ref: https://github.com/5g-ucl-idrbt/oai-gnodeb-b210
- Commands to be executed in gNB
Plug in the USRP B210 in USB 3.0 port
```
sudo sysctl net.ipv4.ip_forward=1
sudo iptables -P FORWARD ACCEPT
sudo ip route add 192.168.70.128/26 via <Bridge IP of Core VM>
```
- To run the gNB docker 
```
cd ci-scripts/yaml_files/sa_b200_gnb/
sudo docker-compose up -d
```
- To get into the gNB shell
```
sudo docker exec -it sa-b200-gnb bash
```
- Execute the commands to run the gNB
```
bash bin/entrypoint.sh
/opt/oai-gnb/bin/nr-softmodem -O /opt/oai-gnb/etc/gnb.conf $USE_ADDITIONAL_OPTIONS
```
## Testing The Slice
- Now the very first UE device which latches to the network will latch to the banking security slice. It can be configured at ```5G-ovs-integration/docker-compose/ryuctrlr/automac_UEbind.py``` at ```LINE:80```
- On the first UE device open a browser and go the url http://10.0.0.3:3000 you will be able to get the website and you can use the credentials to check account number: ```713047``` and password: ```abhi123```
- Now connect the 2nd UE to the network and try to go to the same url, you will see that the 2nd UE will not fetch the website.
- Due to slicing we have isolated the 1st UE with the access to the banking portal website
## Observation
Even if the server is being hosted on 10.0.0.2:3000 the UE is able to access the server via 10.0.0.3:3000 which is the ip of the router which is going towards the internet. Here, we have isolated the server on the network layer level.
## To down the setup
In Core pc
```
cd 5G-ovs-integration/docker-compose
sudo docker compose -f docker-compose-slicing-bank-nrf.yaml down
```
In gNB PC
```
cd oai-gnodeb/ci-scripts/yaml_files/sa_b200_gnb/
sudo docker-compose down
```





Issues:
------------
Whenever UE connected But not getting Internet.

In UE:
```Go to settings > Access point Names > Update APN ```

For APN Name go to oai_db3.sql file which located in database and check the values of SST and SD. Then go to docker-compose-basic-nrf.yaml file and check  SST and SD values in oai-amf section according to that values check DNN_NI in oai-smf section. Here the DNN_NI Name is APN. Update and reconnect we will get Internet.
