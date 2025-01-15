##### <u>How does a TCP Packet look like?</u>
- ![[Pasted image 20250105140843.png]]
- The TCP Packet is responsible to sending the HTTP data over the destination, for example if a GET Request is sent, then you that info would be embedded in the Data Section of a TCP Packet
- A TCP Packet is send from source IP Address to Destination IP Address
 
##### <u>How does the TCP algo look like?</u>
- To establish connection with a server, a 3 way handshake is done
	- SYN with ISN (initial sequence number) is sent to the server
	- SYN-ACK with another ISN is sent to the client
	- ACK is send by the client to the server
- After this the data is transmitted in form of TCP Packets

##### <u>How does the ACK and Sequence number work?</u>
- When a device (client or server) sends data, it includes a **Sequence Number** in the TCP header.
- The receiver sends back an **ACK packet** with an **ACK Number**, which is the **next expected sequence number**.
- This tells the sender that all previous bytes up to **(ACK Number - 1)** have been received successfully.
###### **Example of ACK Number in Action**
###### **Step 1: Initial Data Transmission**
- **Client → Server**: Sends **100 bytes** with **Sequence Number = 1000**.
- **Data Bytes Sent**: `1000 to 1099`.
###### **Step 2: Acknowledgment from the Server**
- **Server → Client**: Sends an **ACK packet** with:
    - **ACK Number = 1100** (Next expected byte)
    - This means the server successfully received **bytes 1000–1099**.
##### **Step 3: Next Data Transmission**
- **Client → Server**: Sends **another 200 bytes** with **Sequence Number = 1100**.
- **Data Bytes Sent**: `1100 to 1299`.
##### **Step 4: Acknowledgment from the Server**
- **Server → Client**: Sends an **ACK Number = 1300** (confirming receipt of bytes 1100–1299).
##### <u>Why is the ACK Number Important?</u>
- When a device (client or server) sends data, it includes a **Sequence Number** in the TCP header.
1. **Reliable Data Delivery** – Ensures no data is lost by confirming received segments.
2. **Flow Control** – Works with TCP’s **Sliding Window Mechanism** to regulate data transmission speed.
3. **Out-of-Order Handling** – If packets arrive out of order, TCP can request retransmissions based on missing sequence numbers.