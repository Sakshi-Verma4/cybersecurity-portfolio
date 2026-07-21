# cybersecurity-portfolio

# Home Lab Setup - Kali linux + Ubuntu Network

#Objective
Build an isolated lab environment to practice offensive and defensive security techniques safely, without affecting any real network.

##setup
- Attacker machine: Kali Linux (Virtual Box)
- Target machine: ubuntu server (Virtual Box)
- Network mode: Internal network (Isolated from host and internet)
- IP addressing: Manually assigned static IPs on both VMs since Internal Network mode has no DHCP server.

##Commands Used

#Assign static IP on Ubuntu
sudo ip addr add 192.168.56.10/24 dev epn0s3
sudo ip link set enp0s3 up

#assign static IP on Kali
sudo ip addr add 192.168.56.20/24 dev eth0
sudo ip link set eth0 up

# Verify connectivity
ping 192.168.56.10

##Result 
Confirmed succesbetween both VMs via ICMP (ping), establishing a working isolated lab network for further security testing.

##What I learned
-Internet Network mode isolates VMs from the host/Internet but requires manual IP configuration since there's no DHCP
- Manually assigned IPs do not persist across reboots - A real world reminder of why static IP configuration or DHCP reservations matter in production environments.
