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
