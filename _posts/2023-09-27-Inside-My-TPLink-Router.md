---
layout: post
author: jack
category: [hardware]
--- 

Recently my TPLink router, AC750 Dual band, at my place got fried because of a faulty power connection in my switch board. I got a new one and obviously tore apart the old one to see what's inside. That was the first time I ever saw inside a router/access point system, so it was quite interesting what's inside that plastic box. Once I opened it there were three wires each entering into one of the three antennas. Out of three, one had this removable pin thing and other two were soldered. After seperating the board from the antenna wires, I was finally able to look at each part on the board.

<img src="/assets/images/tplink-components-inside.png" width="50%" height="50%">

I have never seen the components of a router but I was quite surprised that it has a very few components with easily understandable sections like power management unit, main SoC, RAM and BIOS. Below is a brief description of each label in the above image.

1. This is the power management section with a 470uF capacitor and one LM5069 current controller IC along with a bunch of resistors and a timer circuit. This section is near to the power jack and the main ON/OFF push button.

2. This is the BIOS section with a cfeon flash memory [https://www.epal.pk/product/cfeon-q32b-flash-memory-chip-32mbit-4mb/](https://www.epal.pk/product/cfeon-q32b-flash-memory-chip-32mbit-4mb/)

3. This is the main SoC of the system. The router is driven by a mediatek MT7628 Wifi Router SoC [https://www.mediatek.com/products/home-networking/mt7628k-n-a](https://www.mediatek.com/products/home-networking/mt7628k-n-a) with two antennas as inputs to this SoC. 

4. This is the memory section of the router system. It is a ESMT M14D5 DDR SDRAM memory chip. This is where the OS for the router is be stored that runs the router system. [https://www.alldatasheet.com/view.jsp?Searchword=M14D5121632A-1.5BG2A](https://www.alldatasheet.com/view.jsp?Searchword=M14D5121632A-1.5BG2A)

5. This is the second wifi chip from Mediatek on the router. This is a MediaTek MT7610E which is a wi-fi single chip. This router supports two frequencies - 2.5GHz and 5GHz. The 5GHz part is coming from this wi-fi single chip.[https://www.mediatek.com/products/broadband-wifi/mt7610e](https://www.mediatek.com/products/broadband-wifi/mt7610e)

6. This section is where all LAN ports are present and it is driven by Ethernet magnetic transformers and LAN filter. Here is a good article on what these transformers are [https://networkengineering.stackexchange.com/questions/29927/what-is-the-purpose-of-an-ethernet-magnetic-transformer-and-how-are-they-used] (https://networkengineering.stackexchange.com/questions/29927/what-is-the-purpose-of-an-ethernet-magnetic-transformer-and-how-are-they-used). These are basically to build up the loss speed (like voltage in electrical transformers) so that they can be transmitted through longer distances via ethernet cables.

There are three antennas (I, II and III) where I and II are the main antennas with 2.5GHz transmission attached to the main Wifi Router Mediatek SoC and the III antenna is connected to a Mediatek Wifi Single-chip SoC with 5GHz transmission.


It was really interesting to look inside the router and understand different sections. Just to get a rough idea on what's happening inside the router (just like any normal system), once the power is turned on, the SoC runs instructions from the BIOS chip and starts the OS from the SDRAM chip. Now I am not sure how the two Mediatek chips are synchronized. And probably if the router doesn't support 5GHz, there might not be this Wifi single chip on the board. But one thing that is bugging me is that I couldn't find the crystal oscilattor (from where this board is getting the clock signal??). Also there is no RF protection or heat sinks inside this tp-link router.

Here are a few teardown videos from YouTube that I found are similar to the above ones - 
1. [https://youtu.be/q_8dpXrEwZI?si=GBYSGTK6agk0abcW](https://youtu.be/q_8dpXrEwZI?si=GBYSGTK6agk0abcW)
2. [https://www.youtube.com/watch?v=bYJ08mtZyfQ](https://www.youtube.com/watch?v=bYJ08mtZyfQ)
3. [https://www.youtube.com/watch?v=yT6HzJ6sl3I](https://www.youtube.com/watch?v=yT6HzJ6sl3I)
