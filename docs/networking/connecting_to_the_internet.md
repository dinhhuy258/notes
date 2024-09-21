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
