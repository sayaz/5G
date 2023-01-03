## 5G OAI (OpenAirInterface) : POWDER OTA (OverTheAir) Lab

Login to POWDER portal: [POWDER](https://www.powderwireless.net/) 

This profile instantiates an experiment for testing OAI 5G with COTS UEs in standalone mode using resources in the POWDER indoor over-the-air (OTA) lab. The indoor OTA lab includes:

- 4x NI X310 SDRs, each with a UBX-160 daughter card occupying channel 0. The TX/RX and RX2 ports on this channel are connected to broadband antennas. The SDRs are connected via fiber to near-edge compute resources.
- 4x Intel NUC compute nodes, each equipped with a Quectel RM500Q-GL 5G module that has been provisioned with a SIM card. The NUCs are also equipped with NI B210 SDRs, but they are not the focus of this profile.

You can find a diagram of the lab layout here: [OTA Lab diagram](https://gitlab.flux.utah.edu/powderrenewpublic/powder-deployment/-/raw/master/diagrams/ota-lab.png)

The following will be deployed:

- Server-class compute node (d430) with a Docker-based OAI 5G Core Network
- Server-class compute node (d740) with OAI 5G gNodeB (fiber connection to 5GCN and an X310)
- Four Intel NUC compute nodes, each with a 5G module and supporting tools

Note: This profile currently requires the use of the 3550-3600 MHz spectrum range and you need an approved reservation for this spectrum in order to use it. It's also strongly recommended that you include the following necessary resources in your reservation to gaurantee their availability at the time of your experiment:

- A d430 compute node to host the core network
- A d740 compute node for the gNodeB
- One of the four indoor OTA X310s
- All four indoor OTA NUCs

# Steps to follow

Startup scripts will still be running when your experiment becomes ready. Watch the "Startup" column on the "List View" tab for your experiment and wait until all of the compute nodes show "Finished" before proceeding.

After all startup scripts have finished...

On `cn`:

If you'd like to monitor traffic between the various network functions and the gNodeB, start tshark in a session:

```
sudo tcpdump -i demo-oai   -f "not arp and not port 53 and not host archive.ubuntu.com and not host security.ubuntu.com" -w \Users\sayazm\file.pcap
```

In another session, start the 5G core network services. It will take several seconds for the services to start up. Make sure the script indicates that the services are healthy before moving on.

```
cd /var/tmp/oai-cn5g-fed/docker-compose
sudo python3 ./core-network.py --type start-mini --fqdn no --scenario 1
```

In yet another session, start following the logs for the AMF. This way you can see when the UE syncs with the network.
```
sudo docker logs -f oai-amf
```

On `nodeb`:
```
sudo /var/tmp/oairan/cmake_targets/ran_build/build/nr-softmodem -E   -O /var/tmp/etc/oai/gnb.sa.band78.fr1.106PRB.usrpx310.conf --sa
```
On `ota-nucX`:
After you've started the gNodeB, you can bring the COTS UE online. First, start the Quectel connection manager:
```
sudo quectel-CM -s oai -4
```
In another session on the same node, bring the UE online:
```
# turn modem on
sudo modemctl full-functionality
```
The UE should attach to the network and pick up an IP address on the wwan interface associated with the module. You'll see the wwan interface name and the IP address in the stdout of the quectel-CM process.

You should now be able to generate traffic in either direction:
```
# from UE to CN traffic gen node (in session on ota-nucX)
ping 192.168.70.135

# from CN traffic generation service to UE (in session on cn node)
sudo docker exec -it oai-ext-dn ping <IP address from quectel-CM>
```
# Trace Analysis
