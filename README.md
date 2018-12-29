# docker-dummyran
Docker packaging of srsLTE+FauxRF to simulate a RAN (UE+eNB) toward an existing EPC.

This is a slight repackaging of the awesome https://github.com/pgorczak/srslte-docker-emulated/ to separate the EPC from the UE and eNB part, so that we can test an existing EPC. All config files for fauxRF are under /config inside the image, and customization happens at run time using the command in docker-compose.yml.

Note that the dummyue container (i.e. a simulated LTE UE/phone) has no other network interface (I am using network_mode=none) in order to make sure the network used is the LTE one. One can use docker API to perform command within the container (i.e docker exec -it dummyue COMMAND).

## configuration

You may want to change the following variables in the docker-compose.yml : 
* physical network interface the macvlan will run on : tap0
* physical IP address of the eNB : 192.168.50.149 and network 192.168.50.0/24
* MCC MNC : change the occurence of 722 07 in the various place
* MME IP : 192.168.50.101
* eNB ID : 18AF1 (101105) (use rax2 if you wanna quickly convert using CLI)
* SIM IMSI,Ki,opc : 722070000000008,c8eba87c1074edd06885cb0486718341,17b6c0157895bcaa1efc1cef55033f5f
* APN : internet

## build and run 

```
docker-compose build
docker-compose up -d 
```

This will run the eNB and then after a couple of seconds the UE will connect over the shared IPC memory.

In my experience the throughput is around 14Mbps and after a while the UE is disconnecting due to various bugs, but this is still quite awesome to simulate an end to end RAN with opensource components. Latency is way higher than the average LTE in a lab (40ms instead of 15ms). 

## known issues

You need to alter the routing within the lteue container dummyue in order to put a default route :
```
docker exec -it dummyue /bin/bash
route add default tun_srsue
ping 8.8.8.8 # this should work now
``` 

