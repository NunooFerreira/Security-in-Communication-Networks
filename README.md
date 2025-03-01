# Security-in-Communication-Networks
This project implements a network security design for a simulated environment divided into three zones: Inside, Outside, and DMZ. It covers the network topology, the implementation of security policies, static routing and NAT configurations, and includes several exercises and a blocking rules script to demonstrate various network security aspects

## Authors
- **Nuno Ferreira** (121758)
- **Miguel Ferreira** (98345)

## Instructors
- Prof. Paulo Salvador (salvador@ua.pt)
- Prof. António Nogueira (nogueira@ua.pt)

## Project Overview
This project focuses on implementing a secure network topology with three main zones: **INSIDE**, **OUTSIDE**, and **DMZ**. The goal is to configure firewalls, load balancers, and routing rules to ensure secure communication between these zones while adhering to a defined security policy.

### Key Components
- **INSIDE Zone**: Represents the internal network with two subnets, including an administrative network.
- **OUTSIDE Zone**: Simulates the external internet, including a subnet for external devices and attackers.
- **DMZ Zone**: Contains servers accessible from both internal and external networks, including web servers and a DNS server.

## Network Topology
The network is divided into three zones:
1. **INSIDE**: Internal network with subnets for administrative and general use.
2. **OUTSIDE**: External network simulating the internet, including a subnet for attackers.
3. **DMZ**: Demilitarized zone with servers accessible from both internal and external networks.

![image](https://github.com/user-attachments/assets/d1ff60f5-843c-467b-aa7b-94955009d462)



## Security Policy
The implemented security policy allows:
- **INSIDE to DMZ**: ICMP, HTTP, HTTPS, SSH, and DNS for administrative users.
- **INSIDE to OUTSIDE**: UDP traffic on ports 5000-6000.
- **OUTSIDE to DMZ**: ICMP and HTTPS traffic, with specific IP blocking for attackers.
- **DMZ to INSIDE/OUTSIDE**: Established and related connections are allowed.

## Configuration Details
### Static Routes and NAT
- **Static Routes**: Configured on routers and load balancers to direct traffic between zones.
- **NAT**: Implemented on firewalls to translate internal IP addresses to a predefined range when traffic exits the network.

### Load Balancing
- **Sticky Connections**: Ensures that traffic between devices is always sent through the same connection.
- **IP Hash Algorithm**: Used to eliminate the need for synchronization between load balancers.

### Firewall Rules
- **INSIDE to DMZ**: Allows ICMP, HTTP, HTTPS, SSH, and DNS for specific IPs.
- **OUTSIDE to DMZ**: Allows ICMP and HTTPS, with IP-based blocking for attackers.
- **DMZ to INSIDE/OUTSIDE**: Allows established and related connections.

## Services
- **INSIDE to DMZ**: Ping (ICMP), HTTP, HTTPS, SSH, and DNS.
- **INSIDE to OUTSIDE**: UDP traffic on ports 5000-6000.
- **OUTSIDE to DMZ**: ICMP and HTTPS.

## Blocking Rules Script
A script is provided to automatically block traffic from identified attacker IPs. The script uses `tcpdump` to capture traffic and applies firewall rules to block malicious IPs.

## Setup Instructions
1. **Network Configuration**:
   - Configure the routers, load balancers, and firewalls as described in the project documentation.
   - Set up static routes and NAT rules on the firewalls.

2. **Load Balancers**:
   - Configure load balancers with sticky connections and the IP Hash algorithm to ensure traffic consistency.

3. **Firewall Rules**:
   - Apply the firewall rules as specified in the project documentation to enforce the security policy.

4. **Testing**:
   - Test the network by sending traffic between the INSIDE, OUTSIDE, and DMZ zones to ensure that the security policy is enforced.
   - Use the provided script to block traffic from attacker IPs and verify that the blocking rules are applied correctly.

---------------------------------------------------------------------------------------------------------------------------------------------
### IP Addressing

#### RouterInside:
- **PC1**: 10.2.2.100/24
- **f0/1**: 10.2.2.10/24
- **f0/0**: 10.1.1.10/24

#### RouterOutside:
- **PC2**: 200.2.2.100/24
- **f0/1**: 200.2.2.10/24
- **f0/0**: 200.1.1.10/24

#### DMZ:
- **192.1.1.100/24**
- **192.1.1.200/24**

#### LB1A:
- **eth0**: 10.1.1.1/24
- **eth1**: 10.0.0.1/24
- **eth2**: 10.0.2.1/24
- **eth3**: 10.0.3.1/24

#### FW1:
- **eth0**: 10.5.3.2/24
- **eth1**: 10.0.3.2/24
- **eth2**: 10.3.3.2/24
- **eth3**: 10.4.3.2/24
- **eth4**: 10.6.1.2/24

#### LB2A:
- **eth0**: 200.1.1.1/24
- **eth1**: 10.5.1.1/24
- **eth2**: 10.5.2.1/24
- **eth3**: 10.5.3.1/24

#### FW2:
- **eth0**: 10.4.2.2/24
- **eth1**: 10.3.2.2/24
- **eth2**: 10.0.2.2/24
- **eth3**: 10.5.2.2/24
- **eth4**: 10.6.2.2/24

#### LB1B:
- **eth0**: 10.1.1.2/24
- **eth1**: 10.0.0.2/24
- **eth2**: 10.3.2.1/24
- **eth3**: 10.3.3.1/24

#### LB2B:
- **eth0**: 200.1.1.2/24
- **eth1**: 10.5.1.2/24
- **eth2**: 10.4.2.1/24
- **eth3**: 10.4.3.1/24

#### LBDMZ:
- **eth0**: 192.1.1.1/24
- **eth1**: 10.6.1.1/24
- **eth2**: 10.6.2.1/24

### Commands Used

### Router 1 Inside:
- conf term
- ip route 0.0.0.0 0.0.0.0 10.1.1.1   # Send all traffic outside through these interfaces
- ip route 0.0.0.0 0.0.0.0 10.1.1.2


### Router 2 Outside:
- conf term
- ip route 10.0.0.0 255.0.0.0 200.1.1.1   # Send all traffic to 10.0.0.0 through LB2A
- ip route 192.1.0.0 255.255.254.0 200.1.1.1   # Send all traffic to DMZ through LB2A

### LB1A:
- set protocols static route 10.2.2.0/24 next-hop 10.1.1.10   # Route for admin network
- set high-availability vrrp group LB1AB_Cluster interface eth1
- set high-availability vrrp group LB1AB_Cluster rfc3768-compatibility
- set high-availability vrrp group LB1AB_Cluster virtual-address 10.0.0.1/24
- set high-availability vrrp group LB1AB_Cluster vrid 1
- set high-availability vrrp sync-group LB1AB_Cluster member LB1AB_Cluster

- set service conntrack-sync accept-protocol tcp,udp,icmp
- set service conntrack-sync failover-mechanism vrrp sync-group LB1AB_Cluster
- set service conntrack-sync interface eth1
- set service conntrack-sync mcast-group 225.0.0.50
- set service conntrack-sync disable-external-cache

- set load-balancing wan interface-health eth2 nexthop 10.0.2.2
- set load-balancing wan interface-health eth3 nexthop 10.0.3.2
- set load-balancing wan rule 1 inbound-interface eth0
- set load-balancing wan rule 1 interface eth2 weight 1
- set load-balancing wan rule 1 interface eth3 weight 1
- set load-balancing wan rule 1 protocol all
- set load-balancing wan sticky-connections inbound
- set load-balancing wan disable-source-nat

  
# NAT Rules
- set nat source rule 10 outbound-interface eth0
- set nat source rule 10 source address 10.0.0.0/8
- set nat source rule 10 translation address 192.1.0.1-192.1.0.10

- set nat source rule 20 outbound-interface eth3
- set nat source rule 20 source address 10.0.0.0/8
- set nat source rule 20 translation address 192.1.0.1-192.1.0.10

# Static Routes
- set protocols static route 0.0.0.0/0 next-hop 10.4.3.1   # Send all traffic outside through eth0
- set protocols static route 0.0.0.0/0 next-hop 10.5.3.1   # Send all traffic outside through eth3
- set protocols static route 10.0.0.0/8 next-hop 10.0.3.1   # Send all traffic to INSIDE through eth1
- set protocols static route 10.0.0.0/8 next-hop 10.3.3.1   # Send all traffic to INSIDE through eth2
- set protocols static route 192.1.1.0/24 next-hop 10.6.1.1   # Send all traffic to DMZ

# Firewall Rules
- set firewall name FROM-DMZ-TO-INSIDE rule 10 action accept
- set firewall name FROM-DMZ-TO-INSIDE rule 10 description "Accept connection DMZ-INSIDE"
- set firewall name FROM-DMZ-TO-INSIDE rule 10 state established enable
- set firewall name FROM-DMZ-TO-INSIDE rule 10 state related enable

- set firewall name FROM-INSIDE-TO-DMZ rule 10 action accept
- set firewall name FROM-INSIDE-TO-DMZ rule 10 description "Accept ICMP only to DMZ"
- set firewall name FROM-INSIDE-TO-DMZ rule 10 destination address 192.1.1.0/24
- set firewall name FROM-INSIDE-TO-DMZ rule 10 icmp type 8
- set firewall name FROM-INSIDE-TO-DMZ rule 10 protocol icmp

- set firewall name FROM-INSIDE-TO-DMZ rule 20 action accept
- set firewall name FROM-INSIDE-TO-DMZ rule 20 description "Accept multiple services for admin use: HTTP, HTTPS, SSH"
- set firewall name FROM-INSIDE-TO-DMZ rule 20 protocol tcp
- set firewall name FROM-INSIDE-TO-DMZ rule 20 destination port 22,80,443
- set firewall name FROM-INSIDE-TO-DMZ rule 20 destination address 192.1.1.0/24
- set firewall name FROM-INSIDE-TO-DMZ rule 20 source address 10.2.2.100/24   # Admin

- set firewall name FROM-INSIDE-TO-DMZ rule 30 action accept
- set firewall name FROM-INSIDE-TO-DMZ rule 30 description "Accept DNS to DMZ"
- set firewall name FROM-INSIDE-TO-DMZ rule 30 destination address 192.1.1.3/24
- set firewall name FROM-INSIDE-TO-DMZ rule 30 protocol udp
- set firewall name FROM-INSIDE-TO-DMZ rule 30 destination port 53
- set firewall name FROM-INSIDE-TO-DMZ rule 30 source address 10.2.2.100/24   # Admin

- set firewall name FROM-INSIDE-TO-DMZ rule 40 action accept
- set firewall name FROM-INSIDE-TO-DMZ rule 40 description "Accept normal use of HTTP and HTTPS for public users"
- set firewall name FROM-INSIDE-TO-DMZ rule 40 destination address 192.1.1.100/24
- set firewall name FROM-INSIDE-TO-DMZ rule 40 destination port 80,443
- set firewall name FROM-INSIDE-TO-DMZ rule 40 protocol tcp







## Conclusion
This project demonstrates the implementation of a secure network topology with defined security policies, load balancing, and firewall rules to protect against external threats while allowing legitimate traffic to flow between zones.
