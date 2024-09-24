
# The Journey of `google.com` After Connecting to Wi-Fi

This explanation outlines how your browser retrieves `google.com` after connecting to a Wi-Fi network, breaking down the process using the OSI model, including DHCP, TCP, HTTP, MAC addresses, NAT, packet breakdown, frame breakdown, routing tables, and DNS lookup.

---

## 1. **Connecting to Wi-Fi and Getting an IP Address (DHCP)**

When you connect to a Wi-Fi network:

1. **DHCP (Dynamic Host Configuration Protocol)** is responsible for assigning your device:
   - **Private IP address** (e.g., `192.168.1.90`)
   - **Subnet mask** (e.g., `255.255.255.0`)
   - **Default gateway** (e.g., `192.168.1.1`)
   - **DNS server IP address** (provided by the ISP)

2. Your computer gets this information from the **Wi-Fi router** using the **DHCP handshake**:
   - **Discover → Offer → Request → Acknowledge**

---

## 2. **DNS Resolution**

1. Your browser needs to resolve the domain `google.com` to an IP address:
   - The **DNS server IP address** was obtained from the DHCP handshake.
   - A **DNS query** is sent to the DNS server (via **UDP** or **TCP**, Layer 4) to request the IP for `google.com`.

2. The ISP's **DNS server** (Layer 7) resolves `google.com` into a **public IP address** (e.g., `142.250.182.14`).

---

## 3. **Establishing a TCP Session with Google's Server**

1. Once the IP address of Google's server is known, your device attempts to establish a **TCP connection** (Layer 4) using a **three-way handshake**:
   - **SYN → SYN-ACK → ACK**
   
2. This handshake occurs between your **private IP** (e.g., `192.168.1.90`) and Google's **public IP**.

3. At this stage, **NAT** on the router will translate your **private IP** into the **public IP** of the router, ensuring that the packets can be routed through the public internet.

---

## 4. **Packet Breakdown & Network Routing (OSI Layer 3)**

### Layer 3: Network Layer (IP Routing and NAT)

1. **Packet Breakdown**:
   - The request to `google.com` is broken into **IP packets** (Layer 3), each containing:
     - **Source IP**: Your device's private IP (initially)
     - **Destination IP**: Google’s public IP address (e.g., `142.250.182.14`).

2. **NAT (Network Address Translation)**:
   - Your Wi-Fi router performs **NAT** at the **Network Layer**:
     - Translates your **private IP** (`192.168.1.90`) to the router's **public IP** (e.g., `136.226.230.91`).
   - This allows your packets to traverse the public internet.

3. **Routing Process**:
   - The router looks up its **routing table** to determine where to forward the packet.
   - It sends the packet to the **ISP’s router** or another device based on the **destination IP**.
   - Each router in the path checks its **routing table** and forwards the packet toward Google's servers.

---

## 5. **Frame Breakdown (OSI Layer 2)**

### Layer 2: Data Link Layer (Frames and MAC Addresses)

1. The **IP packets** are encapsulated into **frames** at the **Data Link Layer**:
   - Each frame includes:
     - **Source MAC address**: Your device’s MAC address (e.g., `00:0c:29:ee:2a:7b`).
     - **Destination MAC address**: The next hop's MAC address (your router’s or ISP’s router’s MAC).

2. The frames are sent across the **Wi-Fi link** using **802.11 protocols** (Layer 2).

---

## 6. **Physical Transmission (OSI Layer 1)**

### Layer 1: Physical Layer

1. The frames are converted into **electrical signals** (or wireless signals in Wi-Fi) and transmitted through the medium (Wi-Fi in this case).
2. These signals travel to your **router**, then onto the **ISP’s network**, and ultimately toward Google's data center.

---

## 7. **NAT and Routing on the Internet**

### NAT at the Network Layer

1. Once the packets leave your local network:
   - **NAT** at the router translates your **source IP** from private (`192.168.1.90`) to public (`136.226.230.91`).
   - **Source port numbers** might also be changed for uniqueness.

2. On the **internet**, routers use their **routing tables** to forward packets based on the **destination IP address** (Google's IP).

3. **Routing Tables**:
   - Each router along the path looks up the **destination IP** in its **routing table** and forwards the packet to the appropriate next hop.

---

## 8. **Returning the Response**

1. **Google's server** receives the request and sends a response.
   - The **destination IP** of this response is your public IP (assigned via NAT).

2. **NAT Translation**:
   - The response arrives at your router with the public IP.
   - The router uses its **NAT table** to translate the **public IP** back into your **private IP** and forwards the response to your device.

3. The **TCP session** is already established, so the data is sent as part of the existing session.

---

## 9. **Displaying the Response**

1. The response packets are reassembled and passed up the **TCP/IP stack**.
2. At the **Application Layer (Layer 7)**, the **HTTP response** is processed by the browser.
3. **Google.com** is rendered on your screen.

---

## Summary of Key Concepts

- **DHCP (Layer 7)**: Assigns IP addresses and other network information when you connect to Wi-Fi.
- **DNS (Layer 7)**: Resolves domain names like `google.com` to IP addresses.
- **NAT (Layer 3)**: Translates private IPs to public IPs so your device can communicate over the internet.
- **Routing Tables (Layer 3)**: Routers use these to forward packets based on destination IPs.
- **MAC Addresses (Layer 2)**: Ensure data is transmitted to the correct physical device on the local network.
- **TCP (Layer 4)**: Establishes reliable connections for data transfer.
- **HTTP (Layer 7)**: The protocol used for web requests, such as loading `google.com`.
