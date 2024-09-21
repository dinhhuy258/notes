# Connecting to the Internet

## The Internet’s Nuts and Bolts

The internet is essentially made of hosts/end systems, communication links, and packet switches.

![](https://natalieagus.github.io/50005//docs/NS/images/01-network-basics/2024-04-23-17-27-17.png) 

### Connecting Modem, Router, Switches, and End Hosts

![](https://natalieagus.github.io/50005//docs/NS/images/01-network-basics/2024-04-24-17-53-03.png) 

**Modem** 

Modem (modulator-demodulator) is the device is a device connecting your home to your internet service provider (ISP), like Vietel, FPT... through a physical connection (fiber, phone line...).
The modem translates the digital 1s and 0s from your computer into analog information for the cable or telephone wire to carry out to the world, and translates incoming analog signals in the same way.
A modem has a single [coaxial port](https://media.angi.com/s3fs-public/White-coax-outlet-208774-.jpg) for the cable connection from your ISP and a single [ethernet port](https://www.lifewire.com/thmb/HLgI2x-7l09P3ku_PhTyPEBhwjY=/1500x0/filters:no_upscale():max_bytes(150000):strip_icc()/ethernet-cable-socket-182148077-57a2244a3df78c3276eec2c6.jpg) to link the Internet port on your router. Modems bring the Internet to your home

**Router** 

Standalone modems aren't able to send data to multiple devices simultaneously. They usually only have one Ethernet port, and only produce one IP address, which identifies your location to the internet. A router connects to all your home's devices (and links them to each other)—through Ethernet cables or Wi-Fi—and then connects to the modem. Your modem receives information from the internet, sends it to the router, and the router sends it to the computer that asked for it.
. The router creates a local area network (LAN) within your house or office. Some modems also include router. Routers bring the Internet to your devices. Router operates at layer 3.

**Access Point** 

Today, though, we have the ability to connect all devices to your home network over Wi-Fi. To do that, you need something to broadcast that wireless signal.
A **wireless access point** connects to your router, usually over Ethernet, and communicates with your Ethernet-less devices over wireless frequencies. Most home users have routers with wireless access points built in, but standalone access points are still common for businesses, since you can pair multiple access points together to extend your network over a large area.

**Switch** 

All routers come with built-in Ethernet ports, but depending on the size and class of router you buy, there may not be enough to plug in all your devices, especially in the age of smart home tech, which often require numerous, hard-wired base stations. If you run out of Ethernet ports on your router, a switch can add more Ethernet ports to your network. You just plug your extra devices into the switch, plug the switch into your router, and they'll appear on your network. Switch operates at layer 2.

Note: you need a router in order to use a switch. A switch can't assign IP addresses or create a network like your router can—it merely acts as a traffic cop for the signals coming through.

## The Internet Hierarchy

![](https://natalieagus.github.io/50005//docs/NS/images/02-internet-hierarchy/2024-04-25-10-30-45.png) 

Internet Service Provider (ISP) is a company which provides internet connection to end user, but there are basically three levels of ISP. There are 3 levels of ISP: Tier-1 ISP, Tier-2 ISP, and Tier-3 ISP. 

**Tier 1 ISPs**

Tier 1 ISPs are the world's highest level of internet service providers. They own and operate the infrastructure that serves as the internet's backbone. These firms have huge networks of high-capacity fibre optic connections that traverse continents and cross seas to connect the world's main data centres. AT&T, Sprint, Level 3 Communications, and Verizon are examples of Tier 1 ISPs.

**Tier 2 ISPs or Regional ISP**

Tier 2 ISPs provides internet services to a specific region or area. They connect Tier 1 networks to individual users or small businesses in different locations. These ISPs typically have smaller networks compared to Tier 1 providers, but they still play an important role in the hierarchy of the ISP network.

**Tier 3 ISPs**

Tier 3 ISPs are the smallest players in the ISP hierarchy, but they play an essential role in providing internet access to local communities. These providers cater to a specific customer base within a limited geographical area that can range from small towns to individual neighborhoods. They offer local connectivity, usually through cable or DSL lines, and are responsible for maintaining their own network infrastructure.

**Internet Exchange Points (IXP)**

An IXP is a physical location similar to a data center, where multiple ISPs and network operators interconnect their networks. By connecting to an IXP, each ISP can exchange traffic with all other ISPs that are connected to the IXP. These connections can be local, regional or international, depending on which ISPs participate.

## The Internet Routing Hierarchy

So how do packets find their way across the Internet? Does every computer connected to the Internet know where the other computers are? Do packets simply get `broadcast` to every computer on the Internet? The answer to both the preceeding questions is `no`. No computer knows where any of the other computers are, and packets do not get sent to every computer. The information used to get packets to their destinations are contained in routing tables kept by each router connected to the Internet.

**Routers are packet switches**. A router is usually connected between networks to route packets between them. Each router knows about it's sub-networks and which IP addresses they use. 

![](https://web.stanford.edu/class/msande91si/www-spr04/readings/week1/InternetWhitepaper_files/ruswp_diag5.gif) 

When a packet arrives at a router, the router examines the IP address put there by the IP protocol layer on the originating computer. The router checks it's routing table. If the network containing the IP address is found, the packet is sent to that network. If the network containing the IP address is not found, then the router sends the packet on a default route, usually up the backbone hierarchy to the next router. Hopefully the next router will know where to send the packet.

If it does not, again the packet is routed upwards until it reaches a NSP backbone. The routers connected to the NSP backbones hold the largest routing tables and here the packet will be routed to the correct backbone, where it will begin its journey downward through smaller and smaller networks until it finds it's destination.
