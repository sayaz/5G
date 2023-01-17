## 5G OAI (OpenAirInterface) : POWDER OTA (OverTheAir) Lab

Login to POWDER portal: [POWDER](https://www.powderwireless.net/) 

From top menu select `Experiments --> Start Experiment`

<img src="https://user-images.githubusercontent.com/22035469/210289571-4959ce4f-89a7-4b06-9fb4-3725256a6f3e.png" width="500" height="150">

Clicking the `Change Profile` button will let you select the profile that your experiment will be built from

<img src="https://user-images.githubusercontent.com/22035469/210289631-f0477fc6-0e54-4291-89dc-66ffe6122b8b.png" width="500" height="160">

From Profile selector (left panel), select `oai-indoor-ota` and click `Select Profile`

<img src="https://user-images.githubusercontent.com/22035469/210289748-374d7541-7449-45cb-b741-0901a836eb43.png" width="600" height="350">

Click on `Show profile` button

<img src="https://user-images.githubusercontent.com/22035469/213006948-060e1fff-7a08-46b4-b3df-e3d7186147a1.png" width="500" height="250">

At the end of the page, `instantiate` from the `test-basic-core` repository

<img src="https://user-images.githubusercontent.com/22035469/213007753-80df7ee4-21d3-4823-8c5b-150484b4085b.png" width="500" height="100">


Select experiment parameters

<img src="https://user-images.githubusercontent.com/22035469/210290134-6d97ca4e-23fc-4ac8-b82d-69cb16e47f61.png" width="500" height="300">

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

Type the `name` and select the `project` to work in and click `Next`

<img src="https://user-images.githubusercontent.com/22035469/210290278-261fced3-8aa4-4927-82bc-f01b0c10d8d0.png" width="500" height="200">

If you already have a previously “approved” reservation, select the checkbox and proceed.

<img src="https://user-images.githubusercontent.com/22035469/210290421-8c6d2ff1-514f-4146-8d8b-e4318ea56a04.png" width="500" height="380">

Provisioning and booting will start and take some time to complete

<img src="https://user-images.githubusercontent.com/22035469/210290500-91dd58a5-9c88-4dc3-93fa-499652676d90.png" width="700" height="200">

Once the provisioning is complete, go to `List View` tab

<img src="https://user-images.githubusercontent.com/22035469/210290582-92e0edbf-63d6-493d-9346-8e139f000a89.png" width="700" height="200">

# Steps to follow

Startup scripts will still be running when your experiment becomes ready. Watch the "Startup" column on the "List View" tab for your experiment and wait until all of the compute nodes show "Finished" before proceeding.

After all startup scripts have finished...

On `cn`:

If you'd like to monitor traffic between the various network functions and the gNodeB, start tshark in a session:

```
sudo tcpdump -i demo-oai   -f "not arp and not port 53 and not host archive.ubuntu.com and not host security.ubuntu.com" -w \Users\userName\fileName.pcap
```

In another session, start the 5G core network services. It will take several seconds for the services to start up. Make sure the script indicates that the services are healthy before moving on.

```
cd /var/tmp/oai-cn5g-fed/docker-compose
sudo python3 ./core-network.py --type start-basic --fqdn no --scenario 1
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
# turn modem off
sudo modemctl airplane-mode

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
To transfer file from CN node to your machine
```
scp userName@ota-nuc1.emulab.net:/users/userName/fileName.pcap /destination/location/fileName.pcap
```
