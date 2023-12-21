# KubePi
Kubernetes on Raspberry PI

For a kubernetes test setup i have been looking at cheap ways to create nodes.
The new raspberry Pi 5 seemed like an ideal candidate.


# Hardware
We will use the following hardware:
- Raspberry Pi 5 8Gb.
- Pimoroni NVME base for Pi 5.
- 1TB M.2 SSD.
- Generic RTL8153 USB3.0 to Gbit Ethernet converter.
- 32GB Class10 uSD card.

# Results
We will try to reach the following results:
- A High Availibility (HA) cluster.
- Booting host OS from uSD card in a lasting way.
- A distributed filesystem based on LongHorn on our M.2 SSD.
- Rancher control website installed on our HA cluster.
- Loadbalancer running on our cluster, mitigating the need for an external one. 

 
