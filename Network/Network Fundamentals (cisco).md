I. the role and function of network components

1. Routers
- Layer 3 of the OSI model
- IP adressing
- Packet forwarding
- NAT (network adress translation)
- Firewall capabilities
![[Routers.png]]

2. Layer 2 and Layer 3 switches
- Layer 2 switches (Data Link Layer) use MAC addresses to forward Ethernet frames within a local network. They create and maintain a MAC address table to track the association between MAC addresses and the switch ports
- Layer 3 switches, also known as multilayer switches, can perform the functions of both a switch and a router. They have the capability to route IP packets between different networks and make forwarding decisions based on IP addresses
(Layer 2 and Layer 3 switches enable efficient and reliable local area network (LAN) communication)

![[Pasted image 20230616152815.png]]
![[Switches.png]]

3. Next-generation Firewalls (NGFWs) and Intrusion Prevention Systems IPS
- NGFWs = traditional firewall functionality + additional security features
- provide deep packet inspection (DPI) = analyze the content of network traffic and apply security policies based on application, user, or content
- NGFWs detect and prevent various types of cyber threats => malware, intrusion attempts, unauthorized access
- IPS = security appliances that monitor network traffic for suspicious patterns and actively block potential threats in real-time

![[NGFWs.png]]
![[IPS schema archi.png]]

4. Access Points (APs)
- Wireless networking devices
- Allow wireless devices to connect to a wired network
- Bridge between wireless devices and network infrastructure
- Can be managed centrally to ensure secure and efficient wireless network operations

![[Access point (LAN).png]]

5. Controllers (Cisco DNA Center and WLC)
- Controllers / DNA (DigitalNetwork Architecture) / WLCs (Wireless LAN Controllers) are network management devices
- Centralized control and configuration of network components
- Manage and coordinate the operation of network devices (like switches and access point)
- Enabling network administrators to automate and streamline network management tasks
- Controllers allow for centralized policy enforcement, security configuration, software updates, and monitoring of network performance

6. Endpoints
- Devices that connect to a network to initiate or consume network services
- They can be computers, laptops, smartphones, tablets, IoT devices, or any device with network connectivity
- Generate and receive data traffic
- Communicate with other network devices
- Utilize network resources and services
- They often require IP addresses, and their security and management are critical to overall network security

7. Servers
- Powerful computers or devices dedicated to providing specific services or resources to client devices on a network
- Handle and respond to client requests efficiently
- Can host applications, websites, databases, files, and other resources
- High-performance hardware, robust storage systems, and specialized software configurations to deliver reliable and scalable services to network users.

![[Servers.png]]

8. PoE (Power over Ethernet)
- Technology that allows network devices to receive power and data through a single Ethernet cable
- Eliminates the need for separate power adapters or electrical outlets for certain network components (like IP phones, wireless access points, and network cameras)
- The Ethernet cable carries both the data signals and the electrical power required to operate the device
- Simplifying installation and reducing clutter

![[PoE.png]]

II.  Characteristics of network topology architectures

1. Two-tier
- Flat network topology
- Two layers : access layer / core layer
- Access Layer : connects end devices (computers, printers, IP phones, to the network)
- Core Layer : handles the high-speed data transfer between different access layer devices

Characteristics:

- Simple and cost-effective design
- Limited scalability and redundancy
- Access layer devices directly connected to the core layer
- Suitable for small to medium-sized networks with low traffic volume

2. Three-tier
- Three layers : access layer / distribution layer / core layer
- Access Layer : connects end devices (computers, printers, IP phones, to the network)
- Distribution Layer : Provides connectivity between access layer devices and the core layer
- Core Layer : handles the high-speed data transfer between different access layer devices

Characteristics:

- Hierarchical structure with three layers
- Access layer connects end devices, distribution layer provides connectivity, and core layer handles high-speed traffic
- Improved scalability, redundancy, and manageability
- Suitable for medium to large-sized networks with moderate to high traffic volume

![[Two-tier Three-tier.png]]

3. Spine-leaf
- Data center network topology architecture
- Designed for high-performance and low-latency environments
- Two layers : spine layer / leaf layer
- Spine Layer : spine switches connected to every leaf switch in a full-mesh topology (providing high-bandwidth connectivity)
- Leaf Layer : connects end devices (servers, storage, systems, switches) to the spine layer

Characteristics:

- Two-layer architecture with spine and leaf layers
- Spine switches provide high-bandwidth connectivity to all leaf switches
- Leaf switches connect end devices to the spine layer
- High scalability, redundancy, and low-latency communication
- Suitable for large data center environments with high-performance requirements

![[Spine-Leaf archi.png]]

4. WAN (Wide Area Network)
- Connects geographically dispersed locations (cities, countries, continents)
- Facilitate communication and data transfer between remote sites
- Utilize leased lines, dedicated connections, or public networks, such as the internet, to establish connectivity
- Involve routers and switches

WANs enable organizations to establish a cohesive network infrastructure across multiple locations, allowing for centralized management and resource sharing.

5. Small office/home office (SOHO)
- Designed for small-scale environments (home offices / small businesses)
- Simple and straightforward
- Providing connectivity for a limited number of devices and users (often use a single router or modem to connect to the internet and provide local network connectivity)

6. On-premise and cloud
- Combination of local network infrastructure and cloud-based services
- On-premise infrastructure : servers, switches, routers that are physically located within the organization's premises
- Cloud services : offer scalable and flexible resources (servers, storage, software aplications)

Organizations can use a hybrid approach, integrating on-premise and cloud resources to optimize their network infrastructure based on specific requirements, security considerations, and cost-effectiveness.

III. Compare physical interface and cabling types

1. Single-mode fiber, multimode fiber, copper
- Single-mode fiber : type of optical fiber / it has a small core diameter (9 microns) / offers higher bandwidth and longer transmission distances
- Multimode fiber : larger core diameter (50 or 62.5 microns) / use for shorter distance (LAN)
- Copper : 

2. Connections (Ethernet shared media and point-to-point)
- Ethernet shared media : common physical medium / multiple devices share th same network segment or medium (using a hub or switch with shared ports) / include techno like 10BASE-T or 10BASE-2
--> less common : they can lea to congestion and collisions (replaced by switched Ethernet networks)
- Point-to-point : involve direct link btwn 2 devices (don't sharing physical medium) --> transmit an receive data (common in modern Ethernet networks) / full duplex / increase network performance

IV. Identify interface and cable issues (collisions, errors, mismatch duplex, and/or speed)
- Collisions : 2 devices transmit data simultaneously, leading to data corruption
--> solution is full-duplex
- Errors : can result from signal degradation / interference / cable issues => data loss / corruption
- Mismatch duplex : 2 devices have different duplex settings => performance problems
- Mismatch speed : 2 connected devices have different data transmission rates => data loss / connectivity issues

V. Campare TCP to UDP
- TCP (Transmission Control Protocol) / UDP (User Datagram Protocol) => both are transport layer protocols in the OSI model
- TCP : provides reliables connection for communication + features (error checking / sequencing / flow control)
--> used for applications (web browsing / email)
- UDP : lightweight communication without connection and features of TCP / use for speed not reliability (like streaming / online gaming)

VI. Configure and verify IPv4 Adressing and Subnetting
- IPv4 => fourth version of the Internet Protocol (using 32-bit adress)
- IPv4 adresses -> divided into network and host portions / Subnetting involves creating smaller networks in a larger one => efficient IP adress allocation
--> Proper IPv4 adressing + subnetting are crucial for efficient IP adresse allocation and netork management

VI. Configure and verify IPv4 adressing and subnetting
- 