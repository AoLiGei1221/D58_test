## Description of the project

Our tool offer a robust and efficient way to perform lan speed tests. The testing tools have been designed to be user-friendly, providing clear metrics and data visualization to facilitate understanding of the network's performance under a variety of conditions.

The results obtained from our testing suite have significant implications. They not only demonstrate the current state of the network's performance but also help in predicting the network's behavior under different loads. This is vital for user to make informed decisions regarding upgrades, maintenance, and configurations tailored to their specific needs.

### LAN Speed Test
Throughout the course of this project, we employed the Internet Control Message Protocol (ICMP) to gauge the speed and responsiveness of Local Area Networks (LANs). This method allowed us to measure the round-trip time for messages sent between the host and a target device on the same network, which is essential for understanding the network's latency characteristics. Later, we came up with a new way by using Transmission Control Protocol (TCP). This not only help with testing ping, we can also send large packet to test the maximum bandwidth.





## Contribution of each member of the team

### A Runyu Yue [Student No: 1007391298]
Define the protocols headers and the check sum. Responsible for the sending logic for the arp and tcp backend(implementation of socket communication without using the "high level socket" but raw socket). Responsible for sending arp request and wrap outside headers to tcp packet and send. Also responsible for writing user interface in python and the plotting of the results.

### B Kaiwei Zhang [Student No: 1007073873]
Responsible for receiving raw data packets, handling ARP reply for address mapping, sending and track ICMP packets to check reply and error reporting, developing a helper function for printing packet headers.

### C Chris (Shaopeng) Lin [Student No: 1006027452]
Responsible for TCP implementation. Develop the TCP front end and used Runyu and Kawei's IP stack to implement **Stop&Wait**, **Fixed Size Sliding Window**, and **Congestion Control Sliding Window**.


## How to run our code
### 1. set up the environment
Mininet VM from class is required.
We need the mininet environment to contain python3
We need the mininet environment to enable x-forwarding
we need the mininet environment to have numpy and matplotlib
numpy: sudo pip3 install numpy
matplotlib: sudo pip3 install matplotlib
Open the terminal of a host in our topology
run make in the src folder to compile our code
run the firsttry.mn in the root directory of image or create your own
open the topology with miniedit

### 2. how to run the ip-ping delay detection:
Go to the src folder
run 
```
python3 input.py --mode IP –-ip <IP we want to send to> –-packet_size <packet size> –-interval <interval> –-num_packets <number of packets>
```
Example command:
```
python3 input.py –mode IP –-ip 10.1.1.2 –-packet_size 100 –-interval 1 –-num_packets 20
```

```shell!
--packet_size: 
    Specifies the number of data bytes to be sent in a packet. The default is 50.
    
--interval: 
    Wait interval seconds between sending each packet. The default is 2.
    
--num_packets: 
    Specifies the number of packet to be sent. The default is 10.
```
In this process the mode, ip field is required, others has default value as described in the ip ping section.

### 3. how to run the tcp delay detection:
Go to the src folder
open up a port in the destination ip to receive packets using 
```
nc -v -n -l <destination port number>
```
example: 
```
nc -v -n -l 101
```
you can also open a iperf server using
```
iperf -s -w <window size> p <destination port>
```
example:
```
iperf -s -w 2m p 103
```
run 
```
python3 input.py -–mode TCP –-ip <destination ip> –-srcport <source port> –-destport <destination port(make sure it is the same port number as the last step)> –-variant <choose from SAW, SWF, SWCC> -–packsize <size for each packet maximum 1460> -–period <period time> 
```

Example command:
```
python3 input.py –-mode TCP –-ip 10.1.1.2 –-srcport 1234 –-destport 101 -–variant SWCC –-packsize 1400 –-period 12
```
```shell!
--srcport               
    The host machine's port we would like to act on. This is required

--destport              
    The destination machine's port for us to connect to. This is required

--mss                   
    Ignore this option. It is there for a placeholder option we once used.
    
--variant               choices=['SAW', 'SWF', 'SWCC']. 
    Specifies three versions of TCP testing we can do. Default 'SAW'.
    
--packsize 
    Specifies the packet size for each segment in integer bytes. Default 1460 bytes

--period
    Specifies the testing period in seconds for the variant. Unused for 'SWF'. Default 10s.
     
```
In this process the mode, ip field is required, source port is required, destination port is required, others has default value as stated in the tcp section

For more information, feel free to use the `-help` option to easily see available options.






### Results we will get:
If the x forwarding is enabled, the picture will be immediately pops up containing the result of the execution

For IP mode, the result is store in result.png
![results](/images/1.png)

For TCP mode
    For SWCC mode, 
    the congestion window plot is in resultConjestionforTcp.png
        the rtt plot is in resultRttforTcp.png
    For the rest of modes,
    the congestion window plot is in resultRttforTcp.png, ther is no conjestion window plot

![RTT](/images/2.png)
![CWND](/images/3.png)

### Sample Stop&Wait:
![Stop&Wait](/images/4.png)

### Sample Fixed Size:
![Fixed Size](/images/5.png)

### Sample Congetsion Controlled:
![SSCC](/images/6.png)


## Implementation details & Documentation
#### RAW Socket implementation:
- int initTCPSocket ()
This function runs on the main function which initialize the raw socket as global variable to send tcp packets
- char * find_active_interface ()
This function returns a pointer to the string of name of the interface the code runs on
- int get_mac_ip (const char *iface, uint8_t *mac, uint32_t *ip)
This function takes in an interface name, put the mac address of the interface to the mac pointer and put the ip of that inter face to the ip pointer
- int send_arp_packet (uint32_t targetIp)
This function send an arp request packet to the targetIp
- int warpHeaderAndSendTcp (uint8_t *tcpbuff, int tcpTotalLen, uint32_t *dest_ip, uint8_t *dest_mac)
This function takes in a buffer which contains tcp packet, the total length of the tcp header and data, the destination ip address, and the destination mac address. This function wraps the tcp packet with ip and ethernet header and send to the destination

#### Python scripts:
- input.py
This file transfers python command line argument and based on the command line argument, it creates different sub processes to run the corresponding mode and options of execution. After generation of data files by these c execution files, this file calles other python scripts which is responsible for data visulization
- plotPing.py
This file plots the rtt of ip mode of the program by reading the generated data file "log". It generates the picture directly in another window. The picture is also stored in the results.png file.
- plottcp.py
This file plots the rtt and conjestion window of the tcp --variant SWCC mode of the program. It generates these 2 pictures directly in other windows by reading the "tcprtt.txt" and "tcpcong.txt". The picture for the rtt plot is stored in resultRttforTcp.png, for the congestion window plot is stored in resultCongestionforTcp.png
- plotTcpWithoutCongestionWindow.py
This file plots the rtt of the tcp mode where the variant is not SWCC. It genertes the picture resultRttforTcp.png by reading the data file named "tcprtt.txt" generated by the system


#### IP Implementation:
- void *packet_receiver(void *arg)
After start running the program, main will first create a thread that keep reveiving packets. 
- uint16_t send_ip_packet (struct arp_header *receive_arp_header, int size);
This function will take received arp_header and generate a icmp echo packet to be send. If successfully sent, it returns identification of icmp packet.
- struct icmp_list
This struct is a linked list that holds all sent icmp packets. It contains a icmp identification and time of packet being sent. After we send a icmp echo packet, we add the new packet to the begining.
- double handle_icmp(unsigned char * buffer, struct icmp_list *head);
After we receive an icmp packet, we will call this function and it comares the receiving imcp identification with what we stored in the imcp_list. If we find corresponding record, we check the end time and return the time between.


#### TCP Implementation:
- `tcp_protocol.h`: __Defines the data structures, constants, and functions to help construct tcp packets__
**Notable Members**:
    - `struct tcp_hdr` : defines the TCP packet header, but without options.
    - `struct tcp_pseudo_hdr` : defines the TCP pseudo-header used in checksum calculation.
    - `void tcp_gen_packet (tcp_hdr_t *header, ...)` : converts host values into the variable header, which can be passed to the network.
    
- `tcp_op.h`: __Defines the TCP procedures such as Handshake and Teardown__
**Notable Members**:
    - `struct tcp_iq tcp_inq` : TAILQ HEAD representing the list of recieved packets
    - `struct tcp_cq tcp_ckq`: TAILQ HEAD representing the list of packets awaiting response
    - `struct tcp_sq tcp_sdq`: TAILQ HEAD representing the list of packets awaiting to be sent
    - `pthread_mutex_t inq_lock` : Syncrhonization between the ckq & inq.
    - `pthread_cond_t inq_cond;` : Used to wake up sending thread when a packet times out or new packets are arriving.
    - `void *tcp_check_timeout ()` : Timer simulation function to check timeouts or other time critical work for keeping track of statistics.
    - `void handle_tcp(...)` : Adds new packets to the tcp_inq list. Uses inq_lock.
    - `uint32_t tcp_handshake(...)` : Does three way handshake
    - `void tcp_stop_and_wait(...)` : Does Stop & Wait indefinitely until the testing period ends
    - `uint32_t tcp_send_sliding_window_fixed(...)` : Sends out fixed window size of packets until the testing period ends.
    - `void tcp_send_sliding_window_slowS_fastR (...)`: Sends out Congestion Controled window size of packets until the testing period ends.
    - `void tcp_teardown` End the connection.
- `tcp_helpers.h`: __Helps initializing Global values and defiens Miscelleneous subprocesses for TCP procedures__
**Notable Members**:
    - `void initializeTCP (...)` : Initializes the required static time values such as lists, locks and user arguments to global.
    - `tcp_hdr_t *tcp_wait_packet (...)` : Blocks and wait for a particular packet. Returns the matched packet.
    - `void tcp_add_sw_packet (...)` : Unblocking call to add a particular packet into the checking phase.
    - `handle_XX_XX_retransmit` : Handles different types of retransmission procedures.
- `tcp_stats.h`: __Defines statistical functions to record data such as RTT.__
**Notable Members**:
    - `void initializeTCP (...)` : Initializes the required static time values such as lists, locks and user arguments to global.
    - `struct tcp_rtt_q tcp_rttQ` : TAILQ HEAD representing the list of RTT values for packets, arrived in the order of iteration.
    - `struct tcp_bw_q tcp_bwQ`: TAILQ HEAD representing the list of bandwidths recorded in the order of insertion.
    - `struct tcp_cong_q tcp_congQ`: TAILQ HEAD representing the list of congestion window recorded in the order of insertion.
## Discussion on Project Result
### Achievements
The project was able to reliabally test the speed in a network and in a low latency network, can achieve a bandwidth up to **400 MBits/s** within Ethernets(1460 bytes only compared to Iperf who sends a almost cheating amount of tens of thousands of bytes per packet). The program not only allows user to find their bandwidth, it also provides insight to optimal window sizes with respect to the bandwidth.
Moreover, testing LAN speed are able to receive the reply packet with similar result compare to real Ping test. It also create a statistic result with a graph of view.
### Difficulties
1. Finding the active interface of the host and get the mac address and ip takes lots of time to investigate
2. handling of global variables is puzzling at first
3. Handling multithreading is difficult especially when the network packet also comes in asynchronously. Finding the correct way to handle in-flight packets and congestion window.

### Drawbacks and Flaws
1. Time contraint forces us to discard Sequence Number Wrapping implementation and Checksum verification. The Checksum isn't too problematic as Mininet don't seem to have a way to corrupt packets? The Wrapping however, would mean if we are stuck with testing lower bandwidth.
2. The congestion control's fast retransmission is not working exactly as expected, potentially due to the network created in Mininet is much more likely to loss packet than recieve and respond.
3. Race condition sometime still persist in the TCP implementation and unfortunately it takes a great deal of work to debug them and we could not offord the time.
4. It is really hard to make the TCP client general purpose and we had to resort to coordinating with particular server implementations.
5. Retransmits indefinitly if the link is  not available.